<script setup>
import { ref } from 'vue'

import Login from './components/Login.vue'
import StudentRegister from './components/StudentRegister.vue'
import StudentDashboard from './components/StudentDashboard.vue'
import ScanQR from './components/ScanQR.vue'
import StudentAttendance from './components/StudentAttendance.vue'
import AdminDashboard from './components/AdminDashboard.vue'

const isLoggedIn = ref(false)
const role = ref('')
const currentPage = ref('dashboard')
const showRegister = ref(false)

const currentStudent = ref(null)

const students = ref(
  JSON.parse(localStorage.getItem('students') || '[]')
)

const events = ref(
  JSON.parse(localStorage.getItem('events') || '[]')
)

const attendanceRecords = ref(
  JSON.parse(localStorage.getItem('attendanceRecords') || '[]')
)

function saveStudents() {
  localStorage.setItem(
    'students',
    JSON.stringify(students.value)
  )
}

function saveEvents() {
  localStorage.setItem(
    'events',
    JSON.stringify(events.value)
  )
}

function saveAttendance() {
  localStorage.setItem(
    'attendanceRecords',
    JSON.stringify(attendanceRecords.value)
  )
}

function login(data) {
  isLoggedIn.value = true
  role.value = data.role
  currentPage.value = 'dashboard'
  showRegister.value = false

  if (data.role === 'student') {
    currentStudent.value = data.student
  }
}

function logout() {
  isLoggedIn.value = false
  role.value = ''
  currentPage.value = 'dashboard'
  currentStudent.value = null
}

function registerStudent(student) {
  const exists = students.value.some(
    s => s.studentId === student.studentId
  )

  if (exists) {
    alert(
      'Student ID already registered. Please login instead.'
    )
    return
  }

  students.value.push(student)

  saveStudents()

  showRegister.value = false

  alert(
    'Account created successfully! Please login.'
  )
}

function addEvent(event) {
  events.value.push(event)

  saveEvents()
}

function deleteEvent(id) {
  events.value = events.value.filter(
    event => event.id !== id
  )

  saveEvents()
}

function recordAttendance(data) {
  const existing = attendanceRecords.value.find(
    record =>
      record.studentId === data.studentId &&
      record.eventCode === data.eventCode
  )

  if (!existing) {
    attendanceRecords.value.unshift({
      studentId: data.studentId,
      studentName: data.studentName,
      gender: data.gender,
      yearLevel: data.yearLevel,
      section: data.section,
      eventName: data.eventName,
      eventCode: data.eventCode,
      timeIn: new Date().toISOString(),
      timeOut: null
    })

    saveAttendance()

    return 'time-in'
  }

  if (!existing.timeOut) {
    existing.timeOut = new Date().toISOString()

    saveAttendance()

    return 'time-out'
  }

  return 'complete'
}
</script>

<template>

  <!-- LOGIN -->
  <Login
    v-if="!isLoggedIn && !showRegister"
    :students="students"
    @login="login"
    @register="showRegister = true"
  />

  <!-- STUDENT REGISTRATION -->
  <StudentRegister
    v-else-if="!isLoggedIn && showRegister"
    @register="registerStudent"
    @back="showRegister = false"
  />

  <!-- ADMIN -->
  <AdminDashboard
    v-else-if="role === 'admin'"
    :events="events"
    :attendance-records="attendanceRecords"
    @add-event="addEvent"
    @delete-event="deleteEvent"
    @logout="logout"
  />

  <!-- STUDENT -->
  <div
    v-else
    class="student-app"
  >

    <!-- SIDEBAR -->
    <aside class="sidebar">

      <!-- LOGO -->
      <div class="logo">

        <div class="logo-icon">
          QR
        </div>

        <div class="logo-text">
          <h2>AttendEase</h2>
          <span>CPSU</span>
        </div>

      </div>

      <!-- NAVIGATION -->
      <nav>

        <button
          :class="{
            active: currentPage === 'dashboard'
          }"
          @click="currentPage = 'dashboard'"
        >
          <span class="nav-icon">🏠</span>
          <span>Dashboard</span>
        </button>

        <button
          :class="{
            active: currentPage === 'scan'
          }"
          @click="currentPage = 'scan'"
        >
          <span class="nav-icon">📷</span>
          <span>QR Scanner</span>
        </button>

        <button
          :class="{
            active: currentPage === 'attendance'
          }"
          @click="currentPage = 'attendance'"
        >
          <span class="nav-icon">📋</span>
          <span>My Attendance</span>
        </button>

      </nav>

      <!-- USER -->
      <div
        v-if="currentStudent"
        class="sidebar-user"
      >

        <div class="user-avatar">
          {{
            currentStudent.name
              ?.charAt(0)
              ?.toUpperCase() || 'S'
          }}
        </div>

        <div class="user-details">
          <strong>
            {{ currentStudent.name }}
          </strong>

          <span>
            {{ currentStudent.studentId }}
          </span>
        </div>

      </div>

      <!-- LOGOUT -->
      <button
        class="logout"
        @click="logout"
      >
        <span>🚪</span>
        <span>Logout</span>
      </button>

    </aside>

    <!-- MAIN CONTENT -->
    <main class="student-content">

      <StudentDashboard
        v-if="currentPage === 'dashboard'"
        :student="currentStudent"
        :attendance-records="attendanceRecords"
      />

      <ScanQR
        v-if="currentPage === 'scan'"
        :student="currentStudent"
        :events="events"
        @attendance-recorded="recordAttendance"
      />

      <StudentAttendance
        v-if="currentPage === 'attendance'"
        :student="currentStudent"
        :attendance-records="attendanceRecords"
      />

    </main>

  </div>

</template>

<style>
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
  font-family: Arial, sans-serif;
}

body {
  background: #f0fdf4;
  color: #1f2937;
}

button {
  font-family: inherit;
}

/* ============================= */
/* STUDENT APP */
/* ============================= */

.student-app {
  min-height: 100vh;
  display: flex;
  background: #f0fdf4;
}

/* ============================= */
/* SIDEBAR */
/* ============================= */

.sidebar {
  width: 255px;
  min-height: 100vh;

  background: #166534;

  color: white;

  padding: 25px 18px;

  display: flex;
  flex-direction: column;

  position: fixed;
  left: 0;
  top: 0;
  bottom: 0;

  z-index: 10;

  box-shadow: 4px 0 15px rgba(0, 0, 0, 0.08);
}

/* ============================= */
/* LOGO */
/* ============================= */

.logo {
  display: flex;
  align-items: center;

  gap: 12px;

  padding: 5px 10px 35px;
}

.logo-icon {
  width: 46px;
  height: 46px;

  border-radius: 13px;

  background: #facc15;

  color: #166534;

  display: flex;
  align-items: center;
  justify-content: center;

  font-weight: 900;
  font-size: 14px;

  box-shadow: 0 5px 12px rgba(0, 0, 0, 0.12);
}

.logo h2 {
  font-size: 20px;
  color: white;
}

.logo span {
  color: #fef08a;
  font-size: 12px;
}

/* ============================= */
/* NAVIGATION */
/* ============================= */

.sidebar nav {
  display: flex;
  flex-direction: column;

  gap: 8px;
}

.sidebar nav button {
  width: 100%;

  border: none;

  background: transparent;

  color: #dcfce7;

  padding: 14px;

  border-radius: 11px;

  text-align: left;

  cursor: pointer;

  font-size: 14px;

  display: flex;
  align-items: center;

  gap: 12px;

  transition: 0.2s;
}

.sidebar nav button:hover {
  background: rgba(255, 255, 255, 0.12);
  color: white;
}

.sidebar nav button.active {
  background: #facc15;
  color: #166534;

  font-weight: 700;

  box-shadow: 0 5px 12px rgba(0, 0, 0, 0.08);
}

.nav-icon {
  width: 25px;

  text-align: center;

  font-size: 17px;
}

/* ============================= */
/* USER AREA */
/* ============================= */

.sidebar-user {
  margin-top: auto;

  padding: 15px;

  background: rgba(255, 255, 255, 0.1);

  border-radius: 13px;

  display: flex;
  align-items: center;

  gap: 10px;

  margin-bottom: 12px;
}

.user-avatar {
  width: 38px;
  height: 38px;

  border-radius: 50%;

  background: #facc15;

  color: #166534;

  display: flex;
  align-items: center;
  justify-content: center;

  font-weight: bold;
}

.user-details {
  min-width: 0;
}

.user-details strong {
  display: block;

  color: white;

  font-size: 12px;

  white-space: nowrap;

  overflow: hidden;

  text-overflow: ellipsis;
}

.user-details span {
  display: block;

  color: #bbf7d0;

  font-size: 10px;

  margin-top: 3px;
}

/* ============================= */
/* LOGOUT */
/* ============================= */

.logout {
  width: 100%;

  border: 1px solid rgba(255, 255, 255, 0.15);

  background: rgba(255, 255, 255, 0.08);

  color: white;

  padding: 13px;

  border-radius: 10px;

  cursor: pointer;

  display: flex;
  align-items: center;

  justify-content: center;

  gap: 8px;

  font-size: 14px;

  transition: 0.2s;
}

.logout:hover {
  background: #dc2626;
  border-color: #dc2626;
}

/* ============================= */
/* MAIN CONTENT */
/* ============================= */

.student-content {
  flex: 1;

  margin-left: 255px;

  min-height: 100vh;

  padding: 35px;

  background: #f0fdf4;
}

/* ============================= */
/* RESPONSIVE */
/* ============================= */

@media (max-width: 768px) {

  .sidebar {
    width: 75px;

    padding: 20px 10px;
  }

  .logo {
    justify-content: center;

    padding-bottom: 30px;
  }

  .logo-text {
    display: none;
  }

  .sidebar nav button {
    justify-content: center;

    padding: 13px 8px;
  }

  .sidebar nav button span:not(.nav-icon) {
    display: none;
  }

  .nav-icon {
    font-size: 19px;
  }

  .sidebar-user {
    justify-content: center;

    padding: 8px;

    background: transparent;
  }

  .user-details {
    display: none;
  }

  .logout span:last-child {
    display: none;
  }

  .logout {
    padding: 13px 8px;
  }

  .student-content {
    margin-left: 75px;

    padding: 20px;
  }
}

@media (max-width: 480px) {

  .sidebar {
    width: 65px;

    padding: 15px 8px;
  }

  .logo {
    padding: 5px 0 25px;
  }

  .logo-icon {
    width: 40px;
    height: 40px;

    font-size: 12px;
  }

  .student-content {
    margin-left: 65px;

    padding: 15px;
  }
}
</style>