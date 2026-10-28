<script setup>
import { ref } from 'vue'

const emit = defineEmits(['register', 'back'])

const studentId = ref('')
const lastName = ref('')
const firstName = ref('')
const middleInitial = ref('')
const gender = ref('')
const program = ref('')
const yearLevel = ref('')
const section = ref('')
const password = ref('')
const confirmPassword = ref('')

const errorMessage = ref('')

function register() {
  errorMessage.value = ''

  if (
    !studentId.value ||
    !lastName.value ||
    !firstName.value ||
    !gender.value ||
    !program.value ||
    !yearLevel.value ||
    !section.value ||
    !password.value ||
    !confirmPassword.value
  ) {
    errorMessage.value = 'Please complete all required fields.'
    return
  }

  if (password.value !== confirmPassword.value) {
    errorMessage.value = 'Passwords do not match.'
    return
  }

  if (password.value.length < 6) {
    errorMessage.value = 'Password must be at least 6 characters.'
    return
  }

  const fullName = middleInitial.value
    ? `${firstName.value} ${middleInitial.value}. ${lastName.value}`
    : `${firstName.value} ${lastName.value}`

  emit('register', {
    studentId: studentId.value,
    name: fullName,
    lastName: lastName.value,
    firstName: firstName.value,
    middleInitial: middleInitial.value,
    gender: gender.value,
    program: program.value,
    yearLevel: yearLevel.value,
    section: section.value,
    password: password.value
  })
}
</script>

<template>
  <div class="register-page">

    <!-- LEFT BRANDING -->
    <section class="brand-panel">

      <div class="brand-content">

        <div class="logo-circle">
          <span>QR</span>
        </div>

        <h1>CPSU</h1>

        <h2>
          Student Attendance<br />
          Monitoring System
        </h2>

        <div class="yellow-line"></div>

        <p>
          Create your student account and start
          managing your university event attendance.
        </p>

      </div>

      <div class="brand-footer">
        Central Philippine State University
      </div>

    </section>


    <!-- REGISTER FORM -->
    <section class="form-panel">

      <div class="register-card">

        <div class="page-title">

          <span class="small-title">
            STUDENT REGISTRATION
          </span>

          <h2>Create Your Account</h2>

          <p>
            Enter your student information below.
          </p>

        </div>


        <form @submit.prevent="register">

          <!-- PERSONAL INFORMATION -->
          <div class="section-title">
            <span class="section-number">01</span>

            <div>
              <h3>Personal Information</h3>
              <p>Enter your basic student details</p>
            </div>
          </div>


          <div class="form-grid">

            <!-- STUDENT ID -->
            <div class="input-group full">
              <label>
                Student ID <span>*</span>
              </label>

              <input
                v-model="studentId"
                type="text"
                placeholder="Enter your Student ID"
              />
            </div>


            <!-- LAST NAME -->
            <div class="input-group">
              <label>
                Last Name <span>*</span>
              </label>

              <input
                v-model="lastName"
                type="text"
                placeholder="Enter last name"
              />
            </div>


            <!-- FIRST NAME -->
            <div class="input-group">
              <label>
                First Name <span>*</span>
              </label>

              <input
                v-model="firstName"
                type="text"
                placeholder="Enter first name"
              />
            </div>


            <!-- MIDDLE INITIAL -->
            <div class="input-group">
              <label>Middle Initial</label>

              <input
                v-model="middleInitial"
                type="text"
                maxlength="1"
                placeholder="M.I."
              />
            </div>


            <!-- GENDER -->
            <div class="input-group">
              <label>
                Gender <span>*</span>
              </label>

              <select v-model="gender">
                <option value="">Select gender</option>
                <option value="Male">Male</option>
                <option value="Female">Female</option>
              </select>
            </div>

          </div>


          <!-- ACADEMIC INFORMATION -->
          <div class="section-title academic">

            <span class="section-number">02</span>

            <div>
              <h3>Academic Information</h3>
              <p>Enter your current academic details</p>
            </div>

          </div>


          <div class="form-grid">

            <!-- PROGRAM -->
            <div class="input-group full">
              <label>
                Program <span>*</span>
              </label>

              <input
                v-model="program"
                type="text"
                placeholder="e.g. BS Information Technology"
              />
            </div>


            <!-- YEAR -->
            <div class="input-group">
              <label>
                Year Level <span>*</span>
              </label>

              <select v-model="yearLevel">
                <option value="">Select year level</option>
                <option value="1st Year">1st Year</option>
                <option value="2nd Year">2nd Year</option>
                <option value="3rd Year">3rd Year</option>
                <option value="4th Year">4th Year</option>
                <option value="5th Year">5th Year</option>
              </select>
            </div>


            <!-- SECTION -->
            <div class="input-group">
              <label>
                Section <span>*</span>
              </label>

              <input
                v-model="section"
                type="text"
                placeholder="e.g. A"
              />
            </div>

          </div>


          <!-- ACCOUNT SECURITY -->
          <div class="section-title academic">

            <span class="section-number">03</span>

            <div>
              <h3>Account Security</h3>
              <p>Create your login password</p>
            </div>

          </div>


          <div class="form-grid">

            <div class="input-group">
              <label>
                Password <span>*</span>
              </label>

              <input
                v-model="password"
                type="password"
                placeholder="Minimum 6 characters"
              />
            </div>


            <div class="input-group">
              <label>
                Confirm Password <span>*</span>
              </label>

              <input
                v-model="confirmPassword"
                type="password"
                placeholder="Re-enter password"
              />
            </div>

          </div>


          <!-- ERROR -->
          <div
            v-if="errorMessage"
            class="error-message"
          >
            <span>!</span>
            {{ errorMessage }}
          </div>


          <!-- BUTTONS -->
          <div class="button-area">

            <button
              type="button"
              class="back-button"
              @click="emit('back')"
            >
              ← Back to Login
            </button>

            <button
              type="submit"
              class="register-button"
            >
              Create Account
              <span>→</span>
            </button>

          </div>


          <p class="required-note">
            <span>*</span> Required fields
          </p>

        </form>

      </div>

    </section>

  </div>
</template>


<style scoped>
* {
  box-sizing: border-box;
}

.register-page {
  min-height: 100vh;
  display: flex;
  background: #f0fdf4;
  font-family: Arial, Helvetica, sans-serif;
}


/* =========================
   LEFT BRANDING
========================= */

.brand-panel {
  width: 35%;
  min-height: 100vh;
  background: #166534;
  color: white;
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
  overflow: hidden;
  padding: 40px;
}

.brand-panel::before {
  content: "";
  position: absolute;
  width: 380px;
  height: 380px;
  border-radius: 50%;
  background: rgba(250, 204, 21, 0.08);
  top: -150px;
  left: -150px;
}

.brand-panel::after {
  content: "";
  position: absolute;
  width: 450px;
  height: 450px;
  border-radius: 50%;
  border: 2px solid rgba(250, 204, 21, 0.12);
  bottom: -250px;
  right: -220px;
}

.brand-content {
  text-align: center;
  max-width: 350px;
  position: relative;
  z-index: 2;
}

.logo-circle {
  width: 90px;
  height: 90px;
  border-radius: 50%;
  background: white;
  border: 6px solid #facc15;
  display: flex;
  align-items: center;
  justify-content: center;
  margin: 0 auto 22px;
  color: #166534;
  font-size: 24px;
  font-weight: 800;
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.18);
}

.brand-content h1 {
  color: #facc15;
  font-size: 45px;
  letter-spacing: 5px;
  margin-bottom: 8px;
}

.brand-content h2 {
  font-size: 21px;
  line-height: 1.4;
  margin-bottom: 20px;
}

.yellow-line {
  width: 60px;
  height: 4px;
  background: #facc15;
  border-radius: 10px;
  margin: 0 auto 20px;
}

.brand-content p {
  color: #dcfce7;
  font-size: 13px;
  line-height: 1.7;
}

.brand-footer {
  position: absolute;
  bottom: 25px;
  width: 100%;
  left: 0;
  text-align: center;
  color: #bbf7d0;
  font-size: 10px;
  z-index: 2;
}


/* =========================
   FORM PANEL
========================= */

.form-panel {
  width: 65%;
  min-height: 100vh;
  padding: 40px 55px;
  display: flex;
  justify-content: center;
  overflow-y: auto;
  background: white;
}

.register-card {
  width: 100%;
  max-width: 800px;
}

.page-title {
  margin-bottom: 25px;
}

.small-title {
  color: #15803d;
  font-size: 11px;
  font-weight: 800;
  letter-spacing: 1.5px;
}

.page-title h2 {
  color: #14532d;
  font-size: 29px;
  margin: 7px 0;
}

.page-title p {
  color: #6b7280;
  font-size: 13px;
}


/* =========================
   SECTION TITLE
========================= */

.section-title {
  display: flex;
  align-items: center;
  gap: 12px;
  margin: 25px 0 15px;
}

.section-title.academic {
  margin-top: 28px;
}

.section-number {
  width: 34px;
  height: 34px;
  border-radius: 9px;
  background: #166534;
  color: #facc15;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 11px;
  font-weight: 800;
}

.section-title h3 {
  color: #14532d;
  font-size: 15px;
}

.section-title p {
  color: #9ca3af;
  font-size: 11px;
  margin-top: 2px;
}


/* =========================
   FORM
========================= */

.form-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 16px;
}

.input-group {
  display: flex;
  flex-direction: column;
  gap: 7px;
}

.input-group.full {
  grid-column: 1 / -1;
}

.input-group label {
  color: #374151;
  font-size: 12px;
  font-weight: 700;
}

.input-group label span,
.required-note span {
  color: #dc2626;
}

.input-group input,
.input-group select {
  width: 100%;
  height: 46px;
  border: 1px solid #d1d5db;
  border-radius: 9px;
  padding: 0 13px;
  outline: none;
  color: #1f2937;
  background: white;
  font-size: 13px;
  transition: 0.2s;
}

.input-group input:focus,
.input-group select:focus {
  border-color: #22c55e;
  box-shadow: 0 0 0 3px rgba(34, 197, 94, 0.10);
}

.input-group input::placeholder {
  color: #9ca3af;
}


/* =========================
   ERROR
========================= */

.error-message {
  margin-top: 18px;
  background: #fef2f2;
  border: 1px solid #fecaca;
  color: #b91c1c;
  border-radius: 9px;
  padding: 11px 13px;
  font-size: 12px;
  display: flex;
  align-items: center;
  gap: 8px;
}

.error-message span {
  width: 18px;
  height: 18px;
  background: #dc2626;
  color: white;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 800;
}


/* =========================
   BUTTONS
========================= */

.button-area {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 15px;
  margin-top: 25px;
}

.back-button {
  border: 1px solid #d1d5db;
  background: white;
  color: #4b5563;
  height: 46px;
  padding: 0 18px;
  border-radius: 9px;
  cursor: pointer;
  font-size: 12px;
  font-weight: 700;
}

.back-button:hover {
  border-color: #166534;
  color: #166534;
}

.register-button {
  border: none;
  background: #166534;
  color: white;
  height: 46px;
  padding: 0 22px;
  border-radius: 9px;
  cursor: pointer;
  font-size: 13px;
  font-weight: 700;
  display: flex;
  align-items: center;
  gap: 10px;
  box-shadow: 0 5px 15px rgba(22, 101, 52, 0.18);
}

.register-button span {
  color: #facc15;
  font-size: 18px;
}

.register-button:hover {
  background: #14532d;
  transform: translateY(-1px);
}

.required-note {
  color: #9ca3af;
  font-size: 10px;
  margin-top: 13px;
}


/* =========================
   RESPONSIVE
========================= */

@media (max-width: 950px) {

  .brand-panel {
    width: 30%;
    padding: 25px;
  }

  .form-panel {
    width: 70%;
    padding: 35px;
  }

  .brand-content h1 {
    font-size: 35px;
  }

  .brand-content h2 {
    font-size: 17px;
  }

}

@media (max-width: 700px) {

  .register-page {
    display: block;
  }

  .brand-panel {
    width: 100%;
    min-height: 220px;
    padding: 25px 20px;
  }

  .logo-circle {
    width: 65px;
    height: 65px;
    border-width: 5px;
    font-size: 18px;
    margin-bottom: 12px;
  }

  .brand-content h1 {
    font-size: 30px;
  }

  .brand-content h2 {
    font-size: 16px;
  }

  .brand-content p,
  .brand-footer {
    display: none;
  }

  .yellow-line {
    width: 45px;
    height: 3px;
    margin-bottom: 0;
  }

  .form-panel {
    width: 100%;
    padding: 30px 20px;
  }

  .form-grid {
    grid-template-columns: 1fr;
  }

  .input-group.full {
    grid-column: auto;
  }

  .button-area {
    flex-direction: column-reverse;
  }

  .back-button,
  .register-button {
    width: 100%;
    justify-content: center;
  }

}
</style>