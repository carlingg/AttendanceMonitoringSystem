<script setup>
import { computed } from 'vue'

const props = defineProps({
  student: {
    type: Object,
    default: null
  },
  attendanceRecords: {
    type: Array,
    default: () => []
  }
})

const myAttendance = computed(() => {
  if (!props.student) return []

  return props.attendanceRecords.filter(
    record => record.studentId === props.student.studentId
  )
})

const totalAttendance = computed(() => {
  return myAttendance.value.length
})

const completedAttendance = computed(() => {
  return myAttendance.value.filter(
    record => record.timeIn && record.timeOut
  ).length
})

const currentlyPresent = computed(() => {
  return myAttendance.value.filter(
    record => record.timeIn && !record.timeOut
  ).length
})

const recentAttendance = computed(() => {
  return [...myAttendance.value]
    .sort((a, b) => {
      return new Date(b.timeIn) - new Date(a.timeIn)
    })
    .slice(0, 5)
})

function formatDate(date) {
  if (!date) return '-'

  return new Date(date).toLocaleDateString('en-US', {
    month: 'short',
    day: 'numeric',
    year: 'numeric'
  })
}

function formatTime(date) {
  if (!date) return '-'

  return new Date(date).toLocaleTimeString('en-US', {
    hour: '2-digit',
    minute: '2-digit'
  })
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
</script>

<template>
  <div class="dashboard">

    <!-- HEADER -->
    <div class="page-header">
      <div>
        <span class="welcome-label">WELCOME BACK</span>

        <h1>
          Hello, {{ student?.name || 'Student' }}!
        </h1>

        <p>
          Here's your attendance overview.
        </p>
      </div>

      <div class="student-badge">
        <div class="student-avatar">
          {{ student?.name?.charAt(0)?.toUpperCase() || 'S' }}
        </div>

        <div>
          <strong>
            {{ student?.name || 'Student' }}
          </strong>

          <span>
            {{ student?.studentId || 'Student ID' }}
          </span>
        </div>
      </div>
    </div>


    <!-- STUDENT INFORMATION -->
    <div class="student-info-card">

      <div class="info-title">
        <div class="title-icon">ID</div>

        <div>
          <h3>Student Information</h3>
          <p>Your registered student details</p>
        </div>
      </div>

      <div class="info-grid">

        <div class="info-item">
          <span>Student ID</span>
          <strong>{{ student?.studentId || '-' }}</strong>
        </div>

        <div class="info-item">
          <span>Full Name</span>
          <strong>{{ student?.name || '-' }}</strong>
        </div>

        <div class="info-item">
          <span>Program</span>
          <strong>{{ student?.program || '-' }}</strong>
        </div>

        <div class="info-item">
          <span>Year Level</span>
          <strong>{{ student?.yearLevel || '-' }}</strong>
        </div>

        <div class="info-item">
          <span>Section</span>
          <strong>{{ student?.section || '-' }}</strong>
        </div>

        <div class="info-item">
          <span>Gender</span>
          <strong>{{ student?.gender || '-' }}</strong>
        </div>

      </div>
    </div>


    <!-- STATISTICS -->
    <div class="stats-grid">

      <div class="stat-card">

        <div class="stat-icon green">
          ✓
        </div>

        <div>
          <span>Total Attendance</span>
          <strong>{{ totalAttendance }}</strong>
        </div>

      </div>


      <div class="stat-card">

        <div class="stat-icon yellow">
          ✓
        </div>

        <div>
          <span>Completed</span>
          <strong>{{ completedAttendance }}</strong>
        </div>

      </div>


      <div class="stat-card">

        <div class="stat-icon light-green">
          ●
        </div>

        <div>
          <span>Currently Present</span>
          <strong>{{ currentlyPresent }}</strong>
        </div>

      </div>

    </div>


    <!-- RECENT ATTENDANCE -->
    <div class="attendance-card">

      <div class="card-header">

        <div>
          <h2>Recent Attendance</h2>
          <p>Your latest event attendance records</p>
        </div>

        <div class="record-count">
          {{ recentAttendance.length }} Records
        </div>

      </div>


      <div
        v-if="recentAttendance.length === 0"
        class="empty-state"
      >

        <div class="empty-icon">
          📋
        </div>

        <h3>No Attendance Records</h3>

        <p>
          Your attendance records will appear here
          after you scan an event QR code.
        </p>

      </div>


      <div
        v-else
        class="table-wrapper"
      >

        <table>

          <thead>
            <tr>
              <th>EVENT</th>
              <th>DATE</th>
              <th>TIME IN</th>
              <th>TIME OUT</th>
              <th>STATUS</th>
            </tr>
          </thead>

          <tbody>

            <tr
              v-for="record in recentAttendance"
              :key="record.eventCode + record.timeIn"
            >

              <td>
                <div class="event-name">
                  <div class="event-icon">
                    QR
                  </div>

                  <strong>
                    {{ record.eventName }}
                  </strong>
                </div>
              </td>

              <td>
                {{ formatDate(record.timeIn) }}
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

  </div>
</template>


<style scoped>
* {
  box-sizing: border-box;
}

.dashboard {
  width: 100%;
  max-width: 1250px;
  margin: 0 auto;
  color: #1f2937;
}


/* =========================
   HEADER
========================= */

.page-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 20px;
  margin-bottom: 25px;
}

.welcome-label {
  color: #15803d;
  font-size: 11px;
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
  font-size: 14px;
}

.student-badge {
  display: flex;
  align-items: center;
  gap: 12px;
  background: white;
  border: 1px solid #dcfce7;
  padding: 10px 15px;
  border-radius: 12px;
  box-shadow: 0 3px 12px rgba(22, 101, 52, 0.06);
}

.student-avatar {
  width: 42px;
  height: 42px;
  border-radius: 50%;
  background: #166534;
  color: #facc15;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 800;
  font-size: 17px;
}

.student-badge div:last-child {
  display: flex;
  flex-direction: column;
  gap: 3px;
}

.student-badge strong {
  color: #14532d;
  font-size: 13px;
}

.student-badge span {
  color: #6b7280;
  font-size: 11px;
}


/* =========================
   STUDENT INFORMATION
========================= */

.student-info-card {
  background: white;
  border: 1px solid #dcfce7;
  border-radius: 16px;
  padding: 22px;
  margin-bottom: 20px;
  box-shadow: 0 4px 15px rgba(22, 101, 52, 0.05);
}

.info-title {
  display: flex;
  align-items: center;
  gap: 12px;
  margin-bottom: 20px;
}

.title-icon {
  width: 42px;
  height: 42px;
  border-radius: 11px;
  background: #166534;
  color: #facc15;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 12px;
  font-weight: 800;
}

.info-title h3 {
  color: #14532d;
  font-size: 17px;
}

.info-title p {
  color: #9ca3af;
  font-size: 12px;
  margin-top: 3px;
}

.info-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 15px;
}

.info-item {
  background: #f0fdf4;
  border-radius: 10px;
  padding: 13px;
  display: flex;
  flex-direction: column;
  gap: 5px;
}

.info-item span {
  color: #6b7280;
  font-size: 11px;
}

.info-item strong {
  color: #14532d;
  font-size: 13px;
}


/* =========================
   STATISTICS
========================= */

.stats-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 18px;
  margin-bottom: 20px;
}

.stat-card {
  background: white;
  border: 1px solid #dcfce7;
  border-radius: 15px;
  padding: 20px;
  display: flex;
  align-items: center;
  gap: 15px;
  box-shadow: 0 4px 15px rgba(22, 101, 52, 0.05);
}

.stat-icon {
  width: 48px;
  height: 48px;
  border-radius: 13px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 18px;
  font-weight: 800;
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

.stat-card div:last-child {
  display: flex;
  flex-direction: column;
  gap: 5px;
}

.stat-card span {
  color: #6b7280;
  font-size: 12px;
}

.stat-card strong {
  color: #14532d;
  font-size: 25px;
}


/* =========================
   ATTENDANCE
========================= */

.attendance-card {
  background: white;
  border: 1px solid #dcfce7;
  border-radius: 16px;
  padding: 22px;
  box-shadow: 0 4px 15px rgba(22, 101, 52, 0.05);
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
  font-size: 18px;
}

.card-header p {
  color: #9ca3af;
  font-size: 12px;
  margin-top: 4px;
}

.record-count {
  background: #fef9c3;
  color: #854d0e;
  border: 1px solid #fde68a;
  padding: 7px 12px;
  border-radius: 20px;
  font-size: 11px;
  font-weight: 700;
}


/* TABLE */

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
  color: #166534;
  font-size: 10px;
  font-weight: 800;
  text-align: left;
  padding: 13px;
  white-space: nowrap;
}

td {
  padding: 14px 13px;
  border-bottom: 1px solid #f0fdf4;
  color: #6b7280;
  font-size: 12px;
  white-space: nowrap;
}

tbody tr:hover {
  background: #fafffb;
}

.event-name {
  display: flex;
  align-items: center;
  gap: 9px;
}

.event-name strong {
  color: #374151;
  font-size: 12px;
}

.event-icon {
  width: 31px;
  height: 31px;
  border-radius: 8px;
  background: #166534;
  color: #facc15;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 9px;
  font-weight: 800;
}


/* STATUS */

.status {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  padding: 6px 9px;
  border-radius: 20px;
  font-size: 10px;
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


/* EMPTY */

.empty-state {
  text-align: center;
  padding: 50px 20px;
}

.empty-icon {
  width: 65px;
  height: 65px;
  border-radius: 50%;
  background: #f0fdf4;
  display: flex;
  align-items: center;
  justify-content: center;
  margin: 0 auto 15px;
  font-size: 25px;
}

.empty-state h3 {
  color: #14532d;
  font-size: 16px;
  margin-bottom: 7px;
}

.empty-state p {
  color: #9ca3af;
  font-size: 12px;
  max-width: 360px;
  margin: auto;
  line-height: 1.6;
}


/* =========================
   RESPONSIVE
========================= */

@media (max-width: 850px) {

  .info-grid {
    grid-template-columns: repeat(2, 1fr);
  }

  .stats-grid {
    grid-template-columns: 1fr;
  }

}

@media (max-width: 650px) {

  .page-header {
    flex-direction: column;
    align-items: flex-start;
  }

  .student-badge {
    width: 100%;
  }

  .info-grid {
    grid-template-columns: 1fr;
  }

  .student-info-card,
  .attendance-card {
    padding: 16px;
  }

  .card-header {
    align-items: flex-start;
    flex-direction: column;
  }

  .page-header h1 {
    font-size: 25px;
  }

}
</style>