<script setup>
import { ref, computed } from 'vue'
import QRCode from 'qrcode'

const props = defineProps({
  events: {
    type: Array,
    default: () => []
  },
  attendanceRecords: {
    type: Array,
    default: () => []
  }
})

const emit = defineEmits([
  'add-event',
  'delete-event',
  'logout'
])

const currentPage = ref('dashboard')

const eventName = ref('')
const eventDate = ref('')
const eventDescription = ref('')

const generatedQR = ref(null)
const generatedEventName = ref('')

const selectedYear = ref('All')
const selectedSection = ref('All')
const selectedDate = ref('All')

const years = computed(() => {
  const values = props.attendanceRecords
    .map(record => record.yearLevel)
    .filter(Boolean)

  return ['All', ...new Set(values)]
})

const sections = computed(() => {
  const values = props.attendanceRecords
    .map(record => record.section)
    .filter(Boolean)

  return ['All', ...new Set(values)]
})

const dates = computed(() => {
  const values = props.attendanceRecords
    .map(record => {
      if (!record.timeIn) return null

      return new Date(record.timeIn)
        .toISOString()
        .split('T')[0]
    })
    .filter(Boolean)

  return ['All', ...new Set(values)]
})

const filteredAttendance = computed(() => {
  return props.attendanceRecords.filter(record => {

    const yearMatch =
      selectedYear.value === 'All' ||
      record.yearLevel === selectedYear.value

    const sectionMatch =
      selectedSection.value === 'All' ||
      record.section === selectedSection.value

    let dateMatch = true

    if (selectedDate.value !== 'All') {
      const recordDate = record.timeIn
        ? new Date(record.timeIn)
            .toISOString()
            .split('T')[0]
        : ''

      dateMatch = recordDate === selectedDate.value
    }

    return yearMatch && sectionMatch && dateMatch
  })
})

const totalScanned = computed(() => {
  return props.attendanceRecords.length
})

const presentToday = computed(() => {
  const today = new Date().toISOString().split('T')[0]

  return props.attendanceRecords.filter(record => {
    if (!record.timeIn) return false

    const date = new Date(record.timeIn)
      .toISOString()
      .split('T')[0]

    return date === today
  }).length
})

const onCampus = computed(() => {
  return props.attendanceRecords.filter(
    record => record.timeIn && !record.timeOut
  ).length
})

const completed = computed(() => {
  return props.attendanceRecords.filter(
    record => record.timeIn && record.timeOut
  ).length
})

const recentAttendance = computed(() => {
  return [...props.attendanceRecords]
    .sort((a, b) => {
      return new Date(b.timeIn) - new Date(a.timeIn)
    })
    .slice(0, 5)
})

function createEvent() {
  if (!eventName.value || !eventDate.value) {
    alert('Please enter the event name and date.')
    return
  }

  const eventCode =
    'CPSU-' +
    Date.now().toString(36).toUpperCase()

  const newEvent = {
    id: Date.now(),
    name: eventName.value,
    date: eventDate.value,
    description: eventDescription.value,
    code: eventCode
  }

  emit('add-event', newEvent)

  eventName.value = ''
  eventDate.value = ''
  eventDescription.value = ''

  alert('Event created successfully!')
}

async function generateQR(event) {
  try {
    const qrDataUrl = await QRCode.toDataURL(event.code, {
      width: 350,
      margin: 2,
      errorCorrectionLevel: 'H'
    })

    generatedQR.value = qrDataUrl
    generatedEventName.value = event.name
  } catch (error) {
    console.error(error)
    alert('Failed to generate QR code.')
  }
}

function downloadQR() {
  if (!generatedQR.value) return

  const link = document.createElement('a')

  link.href = generatedQR.value
  link.download =
    `${generatedEventName.value}-QR.png`

  link.click()
}

function deleteEvent(id) {
  const confirmed = confirm(
    'Are you sure you want to delete this event?'
  )

  if (confirmed) {
    emit('delete-event', id)
  }
}

function closeQR() {
  generatedQR.value = null
  generatedEventName.value = ''
}

function formatDate(date) {
  if (!date) return '-'

  return new Date(date).toLocaleDateString(
    'en-US',
    {
      month: 'short',
      day: 'numeric',
      year: 'numeric'
    }
  )
}

function formatTime(date) {
  if (!date) return '-'

  return new Date(date).toLocaleTimeString(
    'en-US',
    {
      hour: '2-digit',
      minute: '2-digit'
    }
  )
}

function getStatus(record) {
  if (record.timeOut) {
    return 'Completed'
  }

  if (record.timeIn) {
    return 'Present'
  }

  return 'Pending'
}

function exportCSV() {
  if (filteredAttendance.value.length === 0) {
    alert('No attendance records to export.')
    return
  }

  const headers = [
    'Student ID',
    'Full Name',
    'Gender',
    'Year Level',
    'Section',
    'Event',
    'Time In',
    'Time Out',
    'Status'
  ]

  const rows = filteredAttendance.value.map(record => [
    record.studentId,
    record.studentName,
    record.gender,
    record.yearLevel,
    record.section,
    record.eventName,
    record.timeIn
      ? new Date(record.timeIn).toLocaleString()
      : '',
    record.timeOut
      ? new Date(record.timeOut).toLocaleString()
      : '',
    getStatus(record)
  ])

  const csv = [
    headers,
    ...rows
  ]
    .map(row =>
      row
        .map(value =>
          `"${String(value ?? '').replace(/"/g, '""')}"`
        )
        .join(',')
    )
    .join('\n')

  const blob = new Blob([csv], {
    type: 'text/csv;charset=utf-8;'
  })

  const url = URL.createObjectURL(blob)

  const link = document.createElement('a')
  link.href = url
  link.download = 'CPSU-Attendance-Records.csv'
  link.click()

  URL.revokeObjectURL(url)
}
</script>

<template>
  <div class="admin-app">

    <!-- SIDEBAR -->
    <aside class="sidebar">

      <div class="brand">

        <div class="brand-logo">
          QR
        </div>

        <div>
          <h2>AttendEase</h2>
          <span>CPSU ADMIN</span>
        </div>

      </div>


      <nav>

        <button
          :class="{ active: currentPage === 'dashboard' }"
          @click="currentPage = 'dashboard'"
        >
          <span>⌂</span>
          Dashboard
        </button>

        <button
          :class="{ active: currentPage === 'events' }"
          @click="currentPage = 'events'"
        >
          <span>▣</span>
          Events
        </button>

        <button
          :class="{ active: currentPage === 'attendance' }"
          @click="currentPage = 'attendance'"
        >
          <span>☷</span>
          Attendance
        </button>

      </nav>


      <div class="sidebar-bottom">

        <div class="admin-profile">

          <div class="admin-avatar">
            A
          </div>

          <div>
            <strong>Administrator</strong>
            <span>System Admin</span>
          </div>

        </div>


        <button
          class="logout-button"
          @click="emit('logout')"
        >
          <span>↪</span>
          Logout
        </button>

      </div>

    </aside>


    <!-- MAIN -->
    <main class="main-content">

      <!-- =========================
           DASHBOARD
      ========================== -->

      <template v-if="currentPage === 'dashboard'">

        <div class="page-header">

          <div>
            <span class="page-label">
              ADMINISTRATION
            </span>

            <h1>Dashboard</h1>

            <p>
              Monitor student attendance and university events.
            </p>
          </div>

          <button
            class="primary-button"
            @click="currentPage = 'events'"
          >
            + Create Event
          </button>

        </div>


        <!-- STAT CARDS -->
        <div class="stats-grid">

          <div class="stat-card">

            <div class="stat-icon green">
              #
            </div>

            <div>
              <span>Total Scanned</span>
              <strong>{{ totalScanned }}</strong>
            </div>

          </div>


          <div class="stat-card">

            <div class="stat-icon yellow">
              ✓
            </div>

            <div>
              <span>Present Today</span>
              <strong>{{ presentToday }}</strong>
            </div>

          </div>


          <div class="stat-card">

            <div class="stat-icon light-green">
              ●
            </div>

            <div>
              <span>On Campus</span>
              <strong>{{ onCampus }}</strong>
            </div>

          </div>


          <div class="stat-card">

            <div class="stat-icon dark-green">
              ✓
            </div>

            <div>
              <span>Completed</span>
              <strong>{{ completed }}</strong>
            </div>

          </div>

        </div>


        <!-- RECENT -->
        <div class="content-card">

          <div class="card-header">

            <div>
              <h2>Recent Attendance</h2>
              <p>Latest student attendance activity</p>
            </div>

            <button
              class="text-button"
              @click="currentPage = 'attendance'"
            >
              View All →
            </button>

          </div>


          <div
            v-if="recentAttendance.length === 0"
            class="empty-state"
          >
            <div class="empty-icon">
              📋
            </div>

            <h3>No Attendance Yet</h3>

            <p>
              Attendance records will appear here
              after students scan event QR codes.
            </p>
          </div>


          <div
            v-else
            class="table-wrapper"
          >

            <table>

              <thead>
                <tr>
                  <th>STUDENT</th>
                  <th>EVENT</th>
                  <th>YEAR</th>
                  <th>TIME IN</th>
                  <th>TIME OUT</th>
                  <th>STATUS</th>
                </tr>
              </thead>

              <tbody>

                <tr
                  v-for="record in recentAttendance"
                  :key="record.studentId + record.eventCode"
                >

                  <td>
                    <div class="student-cell">

                      <div class="mini-avatar">
                        {{ record.studentName?.charAt(0)?.toUpperCase() }}
                      </div>

                      <div>
                        <strong>
                          {{ record.studentName }}
                        </strong>

                        <span>
                          {{ record.studentId }}
                        </span>
                      </div>

                    </div>
                  </td>

                  <td>
                    {{ record.eventName }}
                  </td>

                  <td>
                    {{ record.yearLevel }}
                  </td>

                  <td>
                    {{ formatTime(record.timeIn) }}
                  </td>

                  <td>
                    {{ formatTime(record.timeOut) }}
                  </td>

                  <td>

                    <span
                      class="status"
                      :class="getStatus(record).toLowerCase()"
                    >
                      <span class="status-dot"></span>
                      {{ getStatus(record) }}
                    </span>

                  </td>

                </tr>

              </tbody>

            </table>

          </div>

        </div>

      </template>


      <!-- =========================
           EVENTS
      ========================== -->

      <template v-if="currentPage === 'events'">

        <div class="page-header">

          <div>
            <span class="page-label">
              EVENT MANAGEMENT
            </span>

            <h1>Events</h1>

            <p>
              Create events and generate QR codes for attendance.
            </p>
          </div>

        </div>


        <!-- CREATE EVENT -->
        <div class="content-card create-card">

          <div class="card-header">

            <div>
              <h2>Create New Event</h2>
              <p>
                Set up an event for student attendance.
              </p>
            </div>

          </div>


          <div class="event-form">

            <div class="form-group">

              <label>Event Name</label>

              <input
                v-model="eventName"
                type="text"
                placeholder="e.g. University Intramurals 2026"
              />

            </div>


            <div class="form-group">

              <label>Event Date</label>

              <input
                v-model="eventDate"
                type="date"
              />

            </div>


            <div class="form-group full">

              <label>Description</label>

              <textarea
                v-model="eventDescription"
                placeholder="Enter event description..."
                rows="3"
              ></textarea>

            </div>


            <button
              class="primary-button create-event"
              @click="createEvent"
            >
              + Create Event
            </button>

          </div>

        </div>


        <!-- EVENT LIST -->
        <div class="content-card">

          <div class="card-header">

            <div>
              <h2>Event List</h2>
              <p>
                {{ events.length }} event(s) created
              </p>
            </div>

          </div>


          <div
            v-if="events.length === 0"
            class="empty-state"
          >

            <div class="empty-icon">
              📅
            </div>

            <h3>No Events Created</h3>

            <p>
              Create your first event above.
            </p>

          </div>


          <div
            v-else
            class="event-list"
          >

            <div
              v-for="event in events"
              :key="event.id"
              class="event-item"
            >

              <div class="event-icon">
                QR
              </div>


              <div class="event-details">

                <h3>
                  {{ event.name }}
                </h3>

                <p>
                  {{ formatDate(event.date) }}
                </p>

                <span>
                  Code: {{ event.code }}
                </span>

              </div>


              <div class="event-actions">

                <button
                  class="qr-button"
                  @click="generateQR(event)"
                >
                  Generate QR
                </button>

                <button
                  class="delete-button"
                  @click="deleteEvent(event.id)"
                >
                  Delete
                </button>

              </div>

            </div>

          </div>

        </div>

      </template>


      <!-- =========================
           ATTENDANCE
      ========================== -->

      <template v-if="currentPage === 'attendance'">

        <div class="page-header">

          <div>
            <span class="page-label">
              ATTENDANCE MONITORING
            </span>

            <h1>Attendance Records</h1>

            <p>
              View and manage student event attendance.
            </p>
          </div>

          <button
            class="export-button"
            @click="exportCSV"
          >
            ↓ Export CSV
          </button>

        </div>


        <!-- FILTERS -->
        <div class="filter-card">

          <div class="filter-title">
            <span>FILTER RECORDS</span>
          </div>

          <div class="filters">

            <div class="filter-group">

              <label>Year Level</label>

              <select v-model="selectedYear">

                <option
                  v-for="year in years"
                  :key="year"
                  :value="year"
                >
                  {{ year }}
                </option>

              </select>

            </div>


            <div class="filter-group">

              <label>Section</label>

              <select v-model="selectedSection">

                <option
                  v-for="section in sections"
                  :key="section"
                  :value="section"
                >
                  {{ section }}
                </option>

              </select>

            </div>


            <div class="filter-group">

              <label>Date</label>

              <select v-model="selectedDate">

                <option
                  v-for="date in dates"
                  :key="date"
                  :value="date"
                >
                  {{ date === 'All'
                    ? 'All Dates'
                    : date
                  }}
                </option>

              </select>

            </div>

          </div>

        </div>


        <!-- ATTENDANCE TABLE -->
        <div class="content-card attendance-card">

          <div class="card-header">

            <div>
              <h2>Student Attendance</h2>
              <p>
                {{ filteredAttendance.length }}
                record(s) found
              </p>
            </div>

          </div>


          <div
            v-if="filteredAttendance.length === 0"
            class="empty-state"
          >

            <div class="empty-icon">
              📋
            </div>

            <h3>No Attendance Records</h3>

            <p>
              No records match the selected filters.
            </p>

          </div>


          <div
            v-else
            class="table-wrapper"
          >

            <table class="attendance-table">

              <thead>

                <tr>
                  <th>STUDENT ID</th>
                  <th>FULL NAME</th>
                  <th>GENDER</th>
                  <th>YEAR</th>
                  <th>SECTION</th>
                  <th>EVENT</th>
                  <th>TIME IN</th>
                  <th>TIME OUT</th>
                  <th>STATUS</th>
                </tr>

              </thead>

              <tbody>

                <tr
                  v-for="record in filteredAttendance"
                  :key="record.studentId + record.eventCode"
                >

                  <td class="student-id">
                    {{ record.studentId }}
                  </td>

                  <td class="name-cell">
                    {{ record.studentName }}
                  </td>

                  <td>
                    {{ record.gender }}
                  </td>

                  <td>
                    {{ record.yearLevel }}
                  </td>

                  <td>
                    {{ record.section }}
                  </td>

                  <td>
                    {{ record.eventName }}
                  </td>

                  <td>
                    {{ formatTime(record.timeIn) }}
                  </td>

                  <td>
                    {{ formatTime(record.timeOut) }}
                  </td>

                  <td>

                    <span
                      class="status"
                      :class="getStatus(record).toLowerCase()"
                    >
                      <span class="status-dot"></span>
                      {{ getStatus(record) }}
                    </span>

                  </td>

                </tr>

              </tbody>

            </table>

          </div>

        </div>

      </template>

    </main>


    <!-- =========================
         QR MODAL
    ========================== -->

    <div
      v-if="generatedQR"
      class="modal-overlay"
      @click.self="closeQR"
    >

      <div class="qr-modal">

        <button
          class="close-modal"
          @click="closeQR"
        >
          ×
        </button>

        <div class="qr-title">

          <span>EVENT QR CODE</span>

          <h2>
            {{ generatedEventName }}
          </h2>

          <p>
            Students can scan this QR code
            to record attendance.
          </p>

        </div>


        <div class="qr-container">

          <img
            :src="generatedQR"
            alt="Event QR Code"
          />

        </div>


        <button
          class="download-button"
          @click="downloadQR"
        >
          ↓ Download QR Code
        </button>

      </div>

    </div>

  </div>
</template>


<style scoped>
* {
  box-sizing: border-box;
}

.admin-app {
  min-height: 100vh;
  display: flex;
  background: #f0fdf4;
  font-family: Arial, Helvetica, sans-serif;
  color: #1f2937;
}


/* =========================
   SIDEBAR
========================= */

.sidebar {
  width: 245px;
  min-height: 100vh;
  background: #166534;
  color: white;
  padding: 25px 17px;
  display: flex;
  flex-direction: column;
  position: fixed;
  left: 0;
  top: 0;
  bottom: 0;
}

.brand {
  display: flex;
  align-items: center;
  gap: 11px;
  padding: 5px 10px 35px;
}

.brand-logo {
  width: 43px;
  height: 43px;
  border-radius: 12px;
  background: white;
  color: #166534;
  border: 3px solid #facc15;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 11px;
  font-weight: 900;
}

.brand h2 {
  font-size: 18px;
}

.brand span {
  color: #bbf7d0;
  font-size: 9px;
  letter-spacing: 1px;
}


.sidebar nav {
  display: flex;
  flex-direction: column;
  gap: 7px;
}

.sidebar nav button {
  border: none;
  background: transparent;
  color: #dcfce7;
  padding: 13px 14px;
  border-radius: 9px;
  text-align: left;
  cursor: pointer;
  font-size: 13px;
  display: flex;
  align-items: center;
  gap: 12px;
  transition: 0.2s;
}

.sidebar nav button span {
  width: 20px;
  text-align: center;
  font-size: 16px;
}

.sidebar nav button:hover {
  background: rgba(255, 255, 255, 0.10);
  color: white;
}

.sidebar nav button.active {
  background: #facc15;
  color: #14532d;
  font-weight: 800;
}


.sidebar-bottom {
  margin-top: auto;
}

.admin-profile {
  border-top: 1px solid rgba(255, 255, 255, 0.12);
  padding: 18px 8px;
  display: flex;
  align-items: center;
  gap: 10px;
}

.admin-avatar {
  width: 35px;
  height: 35px;
  border-radius: 50%;
  background: white;
  color: #166534;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 800;
  font-size: 13px;
}

.admin-profile div:last-child {
  display: flex;
  flex-direction: column;
  gap: 3px;
}

.admin-profile strong {
  font-size: 11px;
}

.admin-profile span {
  color: #bbf7d0;
  font-size: 9px;
}

.logout-button {
  width: 100%;
  border: 1px solid rgba(255,255,255,0.15);
  background: rgba(255,255,255,0.05);
  color: white;
  padding: 11px;
  border-radius: 8px;
  cursor: pointer;
  text-align: left;
  font-size: 12px;
  display: flex;
  align-items: center;
  gap: 10px;
}

.logout-button:hover {
  background: #dc2626;
}


/* =========================
   MAIN
========================= */

.main-content {
  margin-left: 245px;
  width: calc(100% - 245px);
  padding: 35px;
  max-width: 1600px;
}


/* =========================
   PAGE HEADER
========================= */

.page-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 20px;
  margin-bottom: 25px;
}

.page-label {
  color: #15803d;
  font-size: 10px;
  font-weight: 800;
  letter-spacing: 1.5px;
}

.page-header h1 {
  color: #14532d;
  font-size: 30px;
  margin: 6px 0;
}

.page-header p {
  color: #6b7280;
  font-size: 13px;
}


/* =========================
   BUTTONS
========================= */

.primary-button {
  border: none;
  background: #166534;
  color: white;
  padding: 12px 17px;
  border-radius: 9px;
  font-size: 12px;
  font-weight: 700;
  cursor: pointer;
  box-shadow: 0 4px 12px rgba(22,101,52,0.16);
}

.primary-button:hover {
  background: #14532d;
}

.export-button {
  border: 1px solid #bbf7d0;
  background: white;
  color: #166534;
  padding: 11px 16px;
  border-radius: 9px;
  cursor: pointer;
  font-size: 12px;
  font-weight: 700;
}

.export-button:hover {
  background: #f0fdf4;
}


/* =========================
   STATISTICS
========================= */

.stats-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 17px;
  margin-bottom: 20px;
}

.stat-card {
  background: white;
  border: 1px solid #dcfce7;
  border-radius: 14px;
  padding: 19px;
  display: flex;
  align-items: center;
  gap: 14px;
  box-shadow: 0 4px 15px rgba(22,101,52,0.05);
}

.stat-icon {
  width: 47px;
  height: 47px;
  border-radius: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 900;
  font-size: 17px;
}

.stat-icon.green {
  background: #dcfce7;
  color: #15803d;
}

.stat-icon.yellow {
  background: #fef9c3;
  color: #ca8a04;
}

.stat-icon.light-green {
  background: #d1fae5;
  color: #047857;
}

.stat-icon.dark-green {
  background: #166534;
  color: #facc15;
}

.stat-card div:last-child {
  display: flex;
  flex-direction: column;
  gap: 5px;
}

.stat-card span {
  color: #6b7280;
  font-size: 11px;
}

.stat-card strong {
  color: #14532d;
  font-size: 24px;
}


/* =========================
   CONTENT CARD
========================= */

.content-card {
  background: white;
  border: 1px solid #dcfce7;
  border-radius: 15px;
  padding: 22px;
  margin-bottom: 20px;
  box-shadow: 0 4px 15px rgba(22,101,52,0.05);
}

.card-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 15px;
  margin-bottom: 20px;
}

.card-header h2 {
  color: #14532d;
  font-size: 17px;
}

.card-header p {
  color: #9ca3af;
  font-size: 11px;
  margin-top: 4px;
}

.text-button {
  border: none;
  background: transparent;
  color: #15803d;
  font-size: 11px;
  font-weight: 700;
  cursor: pointer;
}


/* =========================
   TABLE
========================= */

.table-wrapper {
  overflow-x: auto;
}

table {
  width: 100%;
  border-collapse: collapse;
}

thead {
  background: #f0fdf4;
}

th {
  padding: 12px;
  color: #166534;
  font-size: 9px;
  text-align: left;
  font-weight: 800;
  white-space: nowrap;
}

td {
  padding: 13px 12px;
  border-bottom: 1px solid #f0fdf4;
  color: #6b7280;
  font-size: 11px;
  white-space: nowrap;
}

tbody tr:hover {
  background: #fafffb;
}

.student-cell {
  display: flex;
  align-items: center;
  gap: 9px;
}

.mini-avatar {
  width: 31px;
  height: 31px;
  border-radius: 50%;
  background: #dcfce7;
  color: #166534;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 800;
  font-size: 11px;
}

.student-cell div:last-child {
  display: flex;
  flex-direction: column;
  gap: 3px;
}

.student-cell strong {
  color: #374151;
  font-size: 11px;
}

.student-cell span {
  color: #9ca3af;
  font-size: 9px;
}


/* =========================
   STATUS
========================= */

.status {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  padding: 5px 9px;
  border-radius: 20px;
  font-size: 9px;
  font-weight: 700;
}

.status-dot {
  width: 6px;
  height: 6px;
  border-radius: 50%;
}

.status.completed {
  background: #dcfce7;
  color: #166534;
}

.status.completed .status-dot {
  background: #16a34a;
}

.status.present {
  background: #fef9c3;
  color: #854d0e;
}

.status.present .status-dot {
  background: #eab308;
}

.status.pending {
  background: #f3f4f6;
  color: #6b7280;
}

.status.pending .status-dot {
  background: #9ca3af;
}


/* =========================
   EVENTS FORM
========================= */

.event-form {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 16px;
}

.form-group {
  display: flex;
  flex-direction: column;
  gap: 7px;
}

.form-group.full {
  grid-column: 1 / -1;
}

.form-group label {
  color: #374151;
  font-size: 11px;
  font-weight: 700;
}

.form-group input,
.form-group textarea {
  width: 100%;
  border: 1px solid #d1d5db;
  border-radius: 9px;
  padding: 11px 13px;
  outline: none;
  font-size: 12px;
  font-family: inherit;
}

.form-group input {
  height: 44px;
}

.form-group textarea {
  resize: vertical;
}

.form-group input:focus,
.form-group textarea:focus {
  border-color: #22c55e;
  box-shadow: 0 0 0 3px rgba(34,197,94,0.10);
}

.create-event {
  justify-self: start;
}


/* =========================
   EVENT LIST
========================= */

.event-list {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.event-item {
  border: 1px solid #dcfce7;
  background: #fafffb;
  border-radius: 12px;
  padding: 15px;
  display: flex;
  align-items: center;
  gap: 14px;
}

.event-icon {
  width: 45px;
  height: 45px;
  border-radius: 11px;
  background: #166534;
  color: #facc15;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 11px;
  font-weight: 900;
  flex-shrink: 0;
}

.event-details {
  flex: 1;
}

.event-details h3 {
  color: #14532d;
  font-size: 14px;
  margin-bottom: 4px;
}

.event-details p {
  color: #6b7280;
  font-size: 11px;
  margin-bottom: 4px;
}

.event-details span {
  color: #9ca3af;
  font-size: 9px;
}

.event-actions {
  display: flex;
  align-items: center;
  gap: 8px;
}

.qr-button {
  border: none;
  background: #facc15;
  color: #14532d;
  padding: 9px 12px;
  border-radius: 7px;
  cursor: pointer;
  font-size: 10px;
  font-weight: 800;
}

.qr-button:hover {
  background: #eab308;
}

.delete-button {
  border: 1px solid #fecaca;
  background: white;
  color: #dc2626;
  padding: 9px 12px;
  border-radius: 7px;
  cursor: pointer;
  font-size: 10px;
  font-weight: 700;
}

.delete-button:hover {
  background: #fef2f2;
}


/* =========================
   FILTER
========================= */

.filter-card {
  background: white;
  border: 1px solid #dcfce7;
  border-radius: 15px;
  padding: 18px;
  margin-bottom: 20px;
  box-shadow: 0 4px 15px rgba(22,101,52,0.05);
}

.filter-title {
  color: #166534;
  font-size: 10px;
  font-weight: 800;
  letter-spacing: 1px;
  margin-bottom: 13px;
}

.filters {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 13px;
}

.filter-group {
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.filter-group label {
  color: #6b7280;
  font-size: 10px;
  font-weight: 700;
}

.filter-group select {
  height: 40px;
  border: 1px solid #d1d5db;
  border-radius: 8px;
  padding: 0 10px;
  background: white;
  color: #374151;
  outline: none;
  font-size: 11px;
}

.filter-group select:focus {
  border-color: #22c55e;
}


/* ATTENDANCE TABLE */

.student-id {
  color: #166534;
  font-weight: 700;
}

.name-cell {
  color: #374151;
  font-weight: 700;
}


/* =========================
   EMPTY
========================= */

.empty-state {
  text-align: center;
  padding: 50px 20px;
}

.empty-icon {
  width: 65px;
  height: 65px;
  background: #f0fdf4;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  margin: 0 auto 14px;
  font-size: 24px;
}

.empty-state h3 {
  color: #14532d;
  font-size: 15px;
  margin-bottom: 6px;
}

.empty-state p {
  color: #9ca3af;
  font-size: 11px;
}


/* =========================
   QR MODAL
========================= */

.modal-overlay {
  position: fixed;
  inset: 0;
  background: rgba(20, 83, 45, 0.65);
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 20px;
  z-index: 1000;
}

.qr-modal {
  width: 100%;
  max-width: 430px;
  background: white;
  border-radius: 18px;
  padding: 28px;
  text-align: center;
  position: relative;
  box-shadow: 0 20px 50px rgba(0,0,0,0.2);
}

.close-modal {
  position: absolute;
  top: 13px;
  right: 15px;
  width: 30px;
  height: 30px;
  border: none;
  border-radius: 50%;
  background: #f0fdf4;
  color: #166534;
  font-size: 20px;
  cursor: pointer;
}

.qr-title span {
  color: #15803d;
  font-size: 9px;
  font-weight: 800;
  letter-spacing: 1.5px;
}

.qr-title h2 {
  color: #14532d;
  font-size: 20px;
  margin: 7px 0;
}

.qr-title p {
  color: #9ca3af;
  font-size: 11px;
  line-height: 1.5;
}

.qr-container {
  width: 280px;
  height: 280px;
  background: white;
  border: 8px solid #f0fdf4;
  border-radius: 12px;
  margin: 22px auto;
  display: flex;
  align-items: center;
  justify-content: center;
}

.qr-container img {
  width: 100%;
  height: 100%;
  object-fit: contain;
}

.download-button {
  width: 100%;
  height: 45px;
  border: none;
  border-radius: 9px;
  background: #166534;
  color: white;
  font-weight: 700;
  cursor: pointer;
  font-size: 12px;
}

.download-button:hover {
  background: #14532d;
}


/* =========================
   RESPONSIVE
========================= */

@media (max-width: 1050px) {

  .stats-grid {
    grid-template-columns: repeat(2, 1fr);
  }

}

@media (max-width: 800px) {

  .sidebar {
    width: 75px;
    padding: 20px 10px;
  }

  .brand {
    justify-content: center;
    padding: 5px 0 30px;
  }

  .brand > div:last-child,
  .admin-profile div:last-child {
    display: none;
  }

  .sidebar nav button {
    justify-content: center;
    font-size: 0;
  }

  .sidebar nav button span {
    font-size: 17px;
  }

  .logout-button {
    justify-content: center;
    font-size: 0;
  }

  .main-content {
    margin-left: 75px;
    width: calc(100% - 75px);
    padding: 25px 20px;
  }

  .filters {
    grid-template-columns: 1fr;
  }

}

@media (max-width: 600px) {

  .page-header {
    align-items: flex-start;
    flex-direction: column;
  }

  .stats-grid {
    grid-template-columns: 1fr;
  }

  .event-form {
    grid-template-columns: 1fr;
  }

  .form-group.full {
    grid-column: auto;
  }

  .event-item {
    align-items: flex-start;
    flex-direction: column;
  }

  .event-actions {
    width: 100%;
  }

  .event-actions button {
    flex: 1;
  }

  .content-card {
    padding: 16px;
  }

}
</style>