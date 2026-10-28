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

  return props.attendanceRecords
    .filter(
      record => record.studentId === props.student.studentId
    )
    .sort(
      (a, b) =>
        new Date(b.timeIn) - new Date(a.timeIn)
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

const presentAttendance = computed(() => {
  return myAttendance.value.filter(
    record => record.timeIn && !record.timeOut
  ).length
})

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
</script>

<template>
  <div class="attendance-page">

    <!-- HEADER -->
    <div class="page-header">
      <div>
        <div class="title-row">
          <div class="title-icon">📋</div>

          <div>
            <h1>My Attendance</h1>
            <p>
              View your attendance records for university events.
            </p>
          </div>
        </div>
      </div>
    </div>

    <!-- STUDENT INFORMATION -->
    <div class="student-card" v-if="student">
      <div class="student-avatar">
        {{ student.name?.charAt(0)?.toUpperCase() || 'S' }}
      </div>

      <div class="student-info">
        <h2>{{ student.name }}</h2>

        <p>
          Student ID:
          <strong>{{ student.studentId }}</strong>
        </p>

        <div class="student-tags">
          <span>
            {{ student.program || 'Program not specified' }}
          </span>

          <span>
            {{ student.yearLevel || 'Year level not specified' }}
          </span>

          <span>
            Section {{ student.section || '-' }}
          </span>
        </div>
      </div>
    </div>

    <!-- STATISTICS -->
    <div class="stats-grid">

      <div class="stat-card">
        <div class="stat-icon green">
          📋
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
          🟢
        </div>

        <div>
          <span>Currently Present</span>
          <strong>{{ presentAttendance }}</strong>
        </div>
      </div>

    </div>

    <!-- ATTENDANCE TABLE -->
    <div class="attendance-card">

      <div class="card-header">
        <div>
          <h2>Attendance History</h2>
          <p>
            Your recorded attendance during university events.
          </p>
        </div>

        <div class="record-count">
          {{ totalAttendance }}
          {{ totalAttendance === 1 ? 'Record' : 'Records' }}
        </div>
      </div>

      <!-- EMPTY STATE -->
      <div
        v-if="myAttendance.length === 0"
        class="empty-state"
      >
        <div class="empty-icon">
          📋
        </div>

        <h3>No Attendance Records</h3>

        <p>
          You don't have any attendance records yet.
          Scan an event QR code to record your attendance.
        </p>
      </div>

      <!-- TABLE -->
      <div
        v-else
        class="table-wrapper"
      >
        <table>
          <thead>
            <tr>
              <th>Event</th>
              <th>Date</th>
              <th>Time In</th>
              <th>Time Out</th>
              <th>Status</th>
            </tr>
          </thead>

          <tbody>
            <tr
              v-for="record in myAttendance"
              :key="record.studentId + '-' + record.eventCode"
            >
              <td>
                <div class="event-name">
                  <div class="event-icon">
                    QR
                  </div>

                  <div>
                    <strong>
                      {{ record.eventName }}
                    </strong>

                    <small>
                      {{ record.eventCode }}
                    </small>
                  </div>
                </div>
              </td>

              <td>
                {{ formatDate(record.timeIn) }}
              </td>

              <td>
                <span class="time time-in">
                  {{ formatTime(record.timeIn) }}
                </span>
              </td>

              <td>
                <span
                  v-if="record.timeOut"
                  class="time time-out"
                >
                  {{ formatTime(record.timeOut) }}
                </span>

                <span
                  v-else
                  class="not-recorded"
                >
                  —
                </span>
              </td>

              <td>
                <span
                  class="status"
                  :class="{
                    completed:
                      getStatus(record) === 'Completed',

                    present:
                      getStatus(record) === 'Present',

                    pending:
                      getStatus(record) === 'Pending'
                  }"
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

.attendance-page {
  width: 100%;
  max-width: 1300px;
  margin: 0 auto;
}

/* HEADER */

.page-header {
  margin-bottom: 25px;
}

.title-row {
  display: flex;
  align-items: center;
  gap: 15px;
}

.title-icon {
  width: 55px;
  height: 55px;
  border-radius: 15px;

  background: #166534;
  color: white;

  display: flex;
  align-items: center;
  justify-content: center;

  font-size: 25px;

  box-shadow: 0 6px 15px rgba(22, 101, 52, 0.2);
}

.page-header h1 {
  font-size: 30px;
  color: #166534;
  margin-bottom: 5px;
}

.page-header p {
  color: #6b7280;
  font-size: 14px;
}

/* STUDENT CARD */

.student-card {
  background: white;
  border-radius: 18px;
  padding: 22px;

  display: flex;
  align-items: center;
  gap: 18px;

  border: 1px solid #dcfce7;

  box-shadow: 0 5px 20px rgba(0, 0, 0, 0.05);

  margin-bottom: 20px;
}

.student-avatar {
  width: 65px;
  height: 65px;

  border-radius: 50%;

  background: #166534;
  color: white;

  display: flex;
  align-items: center;
  justify-content: center;

  font-size: 25px;
  font-weight: bold;
}

.student-info h2 {
  color: #1f2937;
  font-size: 20px;
  margin-bottom: 5px;
}

.student-info p {
  color: #6b7280;
  font-size: 14px;
}

.student-info strong {
  color: #166534;
}

.student-tags {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  margin-top: 10px;
}

.student-tags span {
  background: #f0fdf4;
  color: #166534;

  border: 1px solid #bbf7d0;

  padding: 5px 10px;
  border-radius: 20px;

  font-size: 12px;
  font-weight: 600;
}

/* STATISTICS */

.stats-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 18px;

  margin-bottom: 22px;
}

.stat-card {
  background: white;

  border-radius: 16px;

  padding: 20px;

  display: flex;
  align-items: center;
  gap: 15px;

  border: 1px solid #e5e7eb;

  box-shadow: 0 5px 18px rgba(0, 0, 0, 0.04);
}

.stat-icon {
  width: 50px;
  height: 50px;

  border-radius: 13px;

  display: flex;
  align-items: center;
  justify-content: center;

  font-size: 21px;
  font-weight: bold;
}

.stat-icon.green {
  background: #dcfce7;
  color: #166534;
}

.stat-icon.yellow {
  background: #fef9c3;
  color: #ca8a04;
}

.stat-icon.light-green {
  background: #f0fdf4;
  color: #15803d;
}

.stat-card span {
  display: block;

  color: #6b7280;

  font-size: 12px;

  margin-bottom: 5px;
}

.stat-card strong {
  color: #1f2937;
  font-size: 25px;
}

/* ATTENDANCE CARD */

.attendance-card {
  background: white;

  border-radius: 18px;

  border: 1px solid #e5e7eb;

  box-shadow: 0 5px 20px rgba(0, 0, 0, 0.05);

  overflow: hidden;
}

.card-header {
  padding: 22px 24px;

  display: flex;
  align-items: center;
  justify-content: space-between;

  border-bottom: 1px solid #e5e7eb;
}

.card-header h2 {
  color: #166534;
  font-size: 19px;
  margin-bottom: 5px;
}

.card-header p {
  color: #6b7280;
  font-size: 13px;
}

.record-count {
  background: #fef9c3;
  color: #854d0e;

  border: 1px solid #fde68a;

  padding: 8px 13px;

  border-radius: 20px;

  font-size: 12px;
  font-weight: bold;
}

/* TABLE */

.table-wrapper {
  width: 100%;
  overflow-x: auto;
}

table {
  width: 100%;
  border-collapse: collapse;
  min-width: 750px;
}

thead {
  background: #f0fdf4;
}

th {
  text-align: left;

  padding: 15px 18px;

  color: #166534;

  font-size: 12px;
  font-weight: 700;

  text-transform: uppercase;

  border-bottom: 1px solid #dcfce7;
}

td {
  padding: 17px 18px;

  color: #4b5563;

  font-size: 13px;

  border-bottom: 1px solid #f3f4f6;
}

tbody tr:hover {
  background: #fafffb;
}

tbody tr:last-child td {
  border-bottom: none;
}

/* EVENT */

.event-name {
  display: flex;
  align-items: center;
  gap: 10px;
}

.event-icon {
  width: 38px;
  height: 38px;

  border-radius: 10px;

  background: #166534;
  color: white;

  display: flex;
  align-items: center;
  justify-content: center;

  font-size: 10px;
  font-weight: bold;
}

.event-name strong {
  display: block;
  color: #1f2937;
  margin-bottom: 3px;
}

.event-name small {
  color: #9ca3af;
  font-size: 10px;
}

/* TIME */

.time {
  font-weight: 600;
}

.time-in {
  color: #166534;
}

.time-out {
  color: #ca8a04;
}

.not-recorded {
  color: #9ca3af;
}

/* STATUS */

.status {
  display: inline-flex;

  align-items: center;
  gap: 7px;

  padding: 6px 10px;

  border-radius: 20px;

  font-size: 11px;
  font-weight: 700;
}

.status-dot {
  width: 7px;
  height: 7px;

  border-radius: 50%;

  display: inline-block;
}

.status.completed {
  background: #dcfce7;
  color: #166534;
}

.status.completed .status-dot {
  background: #22c55e;
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

/* EMPTY STATE */

.empty-state {
  padding: 70px 30px;

  text-align: center;
}

.empty-icon {
  width: 70px;
  height: 70px;

  border-radius: 20px;

  background: #f0fdf4;

  display: flex;
  align-items: center;
  justify-content: center;

  margin: 0 auto 18px;

  font-size: 30px;
}

.empty-state h3 {
  color: #166534;
  font-size: 19px;
  margin-bottom: 8px;
}

.empty-state p {
  max-width: 450px;
  margin: auto;

  color: #6b7280;

  font-size: 13px;
  line-height: 1.6;
}

/* RESPONSIVE */

@media (max-width: 850px) {
  .stats-grid {
    grid-template-columns: 1fr;
  }

  .student-card {
    align-items: flex-start;
  }
}

@media (max-width: 600px) {
  .page-header h1 {
    font-size: 24px;
  }

  .student-card {
    flex-direction: column;
    text-align: center;
  }

  .student-tags {
    justify-content: center;
  }

  .card-header {
    align-items: flex-start;
    gap: 15px;
    flex-direction: column;
  }

  .student-content {
    padding: 15px;
  }
}
</style>