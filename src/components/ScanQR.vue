<script setup>
import { ref, nextTick, onBeforeUnmount } from 'vue'
import { Html5Qrcode } from 'html5-qrcode'

const props = defineProps({
  student: {
    type: Object,
    default: null
  },
  events: {
    type: Array,
    default: () => []
  }
})

const emit = defineEmits(['attendance-recorded'])

const scanner = ref(null)
const scanning = ref(false)
const cameraError = ref('')
const scanMessage = ref('')
const scanResult = ref('')
const resultType = ref('')

const fileInput = ref(null)

function getAttendanceAction(eventCode) {
  const records = JSON.parse(
    localStorage.getItem('attendanceRecords') || '[]'
  )

  const existing = records.find(
    record =>
      record.studentId === props.student.studentId &&
      record.eventCode === eventCode
  )

  if (!existing) {
    return 'time-in'
  }

  if (!existing.timeOut) {
    return 'time-out'
  }

  return 'complete'
}

async function handleScan(decodedText) {
  if (!decodedText) return

  const code = decodedText.trim()

  const event = props.events.find(
    item => item.code === code
  )

  if (!event) {
    scanMessage.value =
      'Invalid QR code. This event was not found.'
    resultType.value = 'error'
    scanResult.value = ''
    return
  }

  const action = getAttendanceAction(event.code)

  if (action === 'complete') {
    scanMessage.value =
      'You have already completed attendance for this event.'
    resultType.value = 'complete'
    scanResult.value = event.name
    return
  }

  emit('attendance-recorded', {
    studentId: props.student.studentId,
    studentName: props.student.name,
    gender: props.student.gender,
    yearLevel: props.student.yearLevel,
    section: props.student.section,
    eventName: event.name,
    eventCode: event.code
  })

  scanResult.value = event.name

  if (action === 'time-in') {
    scanMessage.value =
      'Time In recorded successfully!'
    resultType.value = 'success'
  } else {
    scanMessage.value =
      'Time Out recorded successfully!'
    resultType.value = 'timeout'
  }

  await stopScanner()
}

async function startScanner() {
  cameraError.value = ''
  scanMessage.value = ''
  scanResult.value = ''
  resultType.value = ''

  await nextTick()

  try {
    scanner.value = new Html5Qrcode('reader')

    await scanner.value.start(
      {
        facingMode: 'environment'
      },
      {
        fps: 10,
        qrbox: {
          width: 250,
          height: 250
        }
      },
      decodedText => {
        handleScan(decodedText)
      },
      () => {}
    )

    scanning.value = true

  } catch (error) {
    console.error(error)

    cameraError.value =
      'Unable to access the camera. Please allow camera permission or use Upload QR.'
  }
}

async function stopScanner() {
  if (scanner.value) {
    try {
      if (scanning.value) {
        await scanner.value.stop()
      }

      await scanner.value.clear()
    } catch (error) {
      console.log(error)
    }

    scanner.value = null
    scanning.value = false
  }
}

function openFilePicker() {
  fileInput.value?.click()
}

async function handleFileUpload(event) {
  const file = event.target.files?.[0]

  if (!file) return

  cameraError.value = ''
  scanMessage.value = ''
  scanResult.value = ''
  resultType.value = ''

  const uploadScannerId = 'qr-image-reader'

  const existingElement =
    document.getElementById(uploadScannerId)

  if (existingElement) {
    existingElement.remove()
  }

  const hiddenReader = document.createElement('div')
  hiddenReader.id = uploadScannerId

  hiddenReader.style.display = 'none'

  document.body.appendChild(hiddenReader)

  const imageScanner = new Html5Qrcode(uploadScannerId)

  try {
    const decodedText =
      await imageScanner.scanFile(file, true)

    await handleScan(decodedText)

  } catch (error) {
    console.error(error)

    cameraError.value =
      'Unable to read the QR code from this image.'

    resultType.value = 'error'

  } finally {
    try {
      await imageScanner.clear()
    } catch (error) {
      console.log(error)
    }

    const element =
      document.getElementById(uploadScannerId)

    if (element) {
      element.remove()
    }

    if (fileInput.value) {
      fileInput.value.value = ''
    }
  }
}

function resetScan() {
  scanMessage.value = ''
  scanResult.value = ''
  resultType.value = ''
  cameraError.value = ''
}

onBeforeUnmount(async () => {
  await stopScanner()
})
</script>


<template>
  <div class="scan-page">

    <!-- HEADER -->
    <div class="page-header">

      <div>
        <span class="page-label">
          ATTENDANCE
        </span>

        <h1>Scan Event QR</h1>

        <p>
          Scan the QR code provided by the event administrator
          to record your attendance.
        </p>
      </div>

    </div>


    <div class="scan-layout">

      <!-- SCANNER CARD -->
      <div class="scanner-card">

        <div class="card-title">

          <div class="title-icon">
            QR
          </div>

          <div>
            <h2>QR Code Scanner</h2>

            <p>
              Position the event QR code inside the frame.
            </p>
          </div>

        </div>


        <!-- CAMERA -->
        <div class="camera-container">

          <div
            id="reader"
            class="reader"
          ></div>

          <div
            v-if="!scanning"
            class="camera-placeholder"
          >

            <div class="camera-icon">
              📷
            </div>

            <h3>Camera Scanner</h3>

            <p>
              Click the button below to activate
              your camera.
            </p>

          </div>

        </div>


        <!-- CAMERA ERROR -->
        <div
          v-if="cameraError"
          class="message error"
        >
          <span>!</span>
          {{ cameraError }}
        </div>


        <!-- RESULT -->
        <div
          v-if="scanMessage"
          class="result-box"
          :class="resultType"
        >

          <div class="result-icon">

            <span v-if="resultType === 'success'">
              ✓
            </span>

            <span v-else-if="resultType === 'timeout'">
              ✓
            </span>

            <span v-else-if="resultType === 'complete'">
              !
            </span>

            <span v-else>
              !
            </span>

          </div>

          <div>

            <strong>
              {{ scanMessage }}
            </strong>

            <p v-if="scanResult">
              Event: {{ scanResult }}
            </p>

          </div>

        </div>


        <!-- CAMERA BUTTONS -->
        <div class="scanner-actions">

          <button
            v-if="!scanning"
            class="start-button"
            @click="startScanner"
          >
            <span>📷</span>
            Start Camera
          </button>

          <button
            v-else
            class="stop-button"
            @click="stopScanner"
          >
            Stop Camera
          </button>

        </div>


        <div class="or-divider">
          <span>OR</span>
        </div>


        <!-- UPLOAD -->
        <button
          class="upload-button"
          @click="openFilePicker"
        >
          <span>↑</span>
          Upload QR Image
        </button>

        <input
          ref="fileInput"
          type="file"
          accept="image/*"
          hidden
          @change="handleFileUpload"
        />


        <button
          v-if="scanMessage"
          class="scan-again-button"
          @click="resetScan"
        >
          Scan Another QR
        </button>

      </div>


      <!-- INSTRUCTIONS -->
      <div class="instructions-card">

        <div class="instructions-header">

          <div class="yellow-icon">
            ?
          </div>

          <div>
            <h2>How to Scan</h2>

            <p>
              Follow these simple steps.
            </p>
          </div>

        </div>


        <div class="steps">

          <div class="step">

            <div class="step-number">
              01
            </div>

            <div>
              <h3>Get the Event QR</h3>

              <p>
                Ask the event administrator for
                the QR code assigned to the event.
              </p>
            </div>

          </div>


          <div class="step">

            <div class="step-number">
              02
            </div>

            <div>
              <h3>Scan the QR Code</h3>

              <p>
                Use your camera and place the QR
                code inside the scanning frame.
              </p>
            </div>

          </div>


          <div class="step">

            <div class="step-number">
              03
            </div>

            <div>
              <h3>Check Your Attendance</h3>

              <p>
                Your Time In or Time Out will be
                automatically recorded.
              </p>
            </div>

          </div>

        </div>


        <!-- TIME IN / TIME OUT -->
        <div class="attendance-info">

          <div class="info-row">

            <div class="info-icon time-in">
              IN
            </div>

            <div>
              <strong>First Scan</strong>

              <span>
                Records your Time In
              </span>
            </div>

          </div>


          <div class="info-row">

            <div class="info-icon time-out">
              OUT
            </div>

            <div>
              <strong>Second Scan</strong>

              <span>
                Records your Time Out
              </span>
            </div>

          </div>

        </div>


        <div class="notice">

          <span>✓</span>

          <p>
            Make sure you are registered before
            scanning an event QR code.
          </p>

        </div>

      </div>

    </div>

  </div>
</template>


<style scoped>
* {
  box-sizing: border-box;
}

.scan-page {
  width: 100%;
  max-width: 1100px;
  margin: 0 auto;
  color: #1f2937;
}


/* =========================
   HEADER
========================= */

.page-header {
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
   LAYOUT
========================= */

.scan-layout {
  display: grid;
  grid-template-columns: minmax(0, 1.25fr) minmax(300px, 0.75fr);
  gap: 20px;
}


/* =========================
   SCANNER CARD
========================= */

.scanner-card,
.instructions-card {
  background: white;
  border: 1px solid #dcfce7;
  border-radius: 16px;
  padding: 22px;
  box-shadow: 0 4px 15px rgba(22, 101, 52, 0.05);
}

.card-title,
.instructions-header {
  display: flex;
  align-items: center;
  gap: 12px;
  margin-bottom: 20px;
}

.title-icon {
  width: 43px;
  height: 43px;
  border-radius: 11px;
  background: #166534;
  color: #facc15;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 11px;
  font-weight: 900;
}

.card-title h2,
.instructions-header h2 {
  color: #14532d;
  font-size: 17px;
}

.card-title p,
.instructions-header p {
  color: #9ca3af;
  font-size: 11px;
  margin-top: 3px;
}


/* =========================
   CAMERA
========================= */

.camera-container {
  position: relative;
  width: 100%;
  min-height: 330px;
  background: #052e16;
  border-radius: 14px;
  overflow: hidden;
  display: flex;
  align-items: center;
  justify-content: center;
}

.reader {
  width: 100%;
  position: relative;
  z-index: 2;
}

.reader :deep(video) {
  width: 100% !important;
  height: 330px !important;
  object-fit: cover !important;
}

.reader :deep(#qr-shaded-region) {
  border-color: #facc15 !important;
}

.camera-placeholder {
  position: absolute;
  inset: 0;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  text-align: center;
  color: white;
  padding: 20px;
  z-index: 1;
}

.camera-icon {
  width: 70px;
  height: 70px;
  border-radius: 50%;
  background: rgba(250, 204, 21, 0.15);
  border: 2px solid #facc15;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 28px;
  margin-bottom: 15px;
}

.camera-placeholder h3 {
  font-size: 16px;
  margin-bottom: 7px;
}

.camera-placeholder p {
  color: #bbf7d0;
  font-size: 11px;
  max-width: 250px;
  line-height: 1.5;
}


/* =========================
   MESSAGES
========================= */

.message {
  margin-top: 15px;
  border-radius: 9px;
  padding: 11px 13px;
  font-size: 11px;
  display: flex;
  align-items: center;
  gap: 9px;
}

.message.error {
  background: #fef2f2;
  color: #b91c1c;
  border: 1px solid #fecaca;
}

.message span {
  width: 19px;
  height: 19px;
  background: #dc2626;
  color: white;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 800;
}


/* =========================
   RESULT
========================= */

.result-box {
  margin-top: 15px;
  padding: 14px;
  border-radius: 11px;
  display: flex;
  align-items: center;
  gap: 12px;
}

.result-icon {
  width: 38px;
  height: 38px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 900;
  flex-shrink: 0;
}

.result-box strong {
  font-size: 12px;
}

.result-box p {
  font-size: 10px;
  margin-top: 4px;
}

.result-box.success,
.result-box.timeout {
  background: #dcfce7;
  color: #166534;
  border: 1px solid #bbf7d0;
}

.result-box.success .result-icon,
.result-box.timeout .result-icon {
  background: #166534;
  color: #facc15;
}

.result-box.complete {
  background: #fef9c3;
  color: #854d0e;
  border: 1px solid #fde68a;
}

.result-box.complete .result-icon {
  background: #facc15;
  color: #854d0e;
}

.result-box.error {
  background: #fef2f2;
  color: #b91c1c;
  border: 1px solid #fecaca;
}

.result-box.error .result-icon {
  background: #dc2626;
  color: white;
}


/* =========================
   BUTTONS
========================= */

.scanner-actions {
  margin-top: 16px;
}

.start-button,
.stop-button,
.upload-button,
.scan-again-button {
  width: 100%;
  height: 45px;
  border-radius: 9px;
  cursor: pointer;
  font-size: 12px;
  font-weight: 700;
  transition: 0.2s;
}

.start-button {
  border: none;
  background: #166534;
  color: white;
  box-shadow: 0 4px 12px rgba(22, 101, 52, 0.15);
}

.start-button:hover {
  background: #14532d;
}

.start-button span {
  margin-right: 7px;
}

.stop-button {
  border: 1px solid #fecaca;
  background: white;
  color: #dc2626;
}

.stop-button:hover {
  background: #fef2f2;
}


/* DIVIDER */

.or-divider {
  display: flex;
  align-items: center;
  gap: 12px;
  margin: 18px 0;
}

.or-divider::before,
.or-divider::after {
  content: "";
  height: 1px;
  background: #e5e7eb;
  flex: 1;
}

.or-divider span {
  color: #9ca3af;
  font-size: 9px;
  font-weight: 700;
}


/* UPLOAD */

.upload-button {
  border: 1px solid #bbf7d0;
  background: #f0fdf4;
  color: #166534;
}

.upload-button:hover {
  background: #dcfce7;
}

.upload-button span {
  color: #ca8a04;
  margin-right: 6px;
}


/* SCAN AGAIN */

.scan-again-button {
  margin-top: 10px;
  border: 1px solid #fde68a;
  background: #fef9c3;
  color: #854d0e;
}

.scan-again-button:hover {
  background: #fef08a;
}


/* =========================
   INSTRUCTIONS
========================= */

.yellow-icon {
  width: 43px;
  height: 43px;
  border-radius: 11px;
  background: #facc15;
  color: #14532d;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 18px;
  font-weight: 900;
}

.steps {
  display: flex;
  flex-direction: column;
  gap: 22px;
  margin-bottom: 22px;
}

.step {
  display: flex;
  gap: 13px;
}

.step-number {
  width: 35px;
  height: 35px;
  border-radius: 9px;
  background: #f0fdf4;
  color: #166534;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 9px;
  font-weight: 900;
  flex-shrink: 0;
}

.step h3 {
  color: #374151;
  font-size: 12px;
  margin-bottom: 5px;
}

.step p {
  color: #9ca3af;
  font-size: 10px;
  line-height: 1.6;
}


/* =========================
   ATTENDANCE INFO
========================= */

.attendance-info {
  border-top: 1px solid #e5e7eb;
  padding-top: 18px;
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.info-row {
  display: flex;
  align-items: center;
  gap: 11px;
}

.info-icon {
  width: 35px;
  height: 35px;
  border-radius: 9px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 8px;
  font-weight: 900;
}

.info-icon.time-in {
  background: #dcfce7;
  color: #166534;
}

.info-icon.time-out {
  background: #fef9c3;
  color: #854d0e;
}

.info-row div:last-child {
  display: flex;
  flex-direction: column;
  gap: 3px;
}

.info-row strong {
  color: #374151;
  font-size: 11px;
}

.info-row span {
  color: #9ca3af;
  font-size: 9px;
}


/* =========================
   NOTICE
========================= */

.notice {
  margin-top: 20px;
  padding: 12px;
  border-radius: 9px;
  background: #f0fdf4;
  display: flex;
  align-items: flex-start;
  gap: 9px;
}

.notice > span {
  width: 18px;
  height: 18px;
  border-radius: 50%;
  background: #166534;
  color: #facc15;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 10px;
  font-weight: 900;
  flex-shrink: 0;
}

.notice p {
  color: #166534;
  font-size: 9px;
  line-height: 1.5;
}


/* =========================
   RESPONSIVE
========================= */

@media (max-width: 850px) {

  .scan-layout {
    grid-template-columns: 1fr;
  }

}

@media (max-width: 600px) {

  .page-header h1 {
    font-size: 25px;
  }

  .scanner-card,
  .instructions-card {
    padding: 16px;
  }

  .camera-container {
    min-height: 280px;
  }

  .reader :deep(video) {
    height: 280px !important;
  }

}
</style>