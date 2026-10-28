<script setup>
import { ref } from 'vue'

const props = defineProps({
  students: {
    type: Array,
    default: () => []
  }
})

const emit = defineEmits(['login', 'register'])

const loginType = ref('student')

const studentId = ref('')
const studentPassword = ref('')

const username = ref('')
const adminPassword = ref('')

const errorMessage = ref('')

function login() {
  errorMessage.value = ''

  // STUDENT LOGIN
  if (loginType.value === 'student') {
    if (!studentId.value || !studentPassword.value) {
      errorMessage.value = 'Please enter your Student ID and password.'
      return
    }

    const student = props.students.find(
      s =>
        s.studentId === studentId.value &&
        s.password === studentPassword.value
    )

    if (!student) {
      errorMessage.value = 'Invalid Student ID or password.'
      return
    }

    emit('login', {
      role: 'student',
      student
    })

    return
  }

  // ADMIN LOGIN
  if (loginType.value === 'admin') {
    if (!username.value || !adminPassword.value) {
      errorMessage.value = 'Please enter your username and password.'
      return
    }

    if (
      username.value === 'admin' &&
      adminPassword.value === 'admin1234'
    ) {
      emit('login', {
        role: 'admin'
      })

      return
    }

    errorMessage.value = 'Invalid admin username or password.'
  }
}

function switchLoginType(type) {
  loginType.value = type
  errorMessage.value = ''

  studentId.value = ''
  studentPassword.value = ''

  username.value = ''
  adminPassword.value = ''
}
</script>

<template>
  <div class="login-page">

    <!-- LEFT BRANDING PANEL -->
    <section class="brand-panel">

      <div class="brand-content">

        <div class="school-logo">
          <div class="logo-circle">
            <span>QR</span>
          </div>
        </div>

        <h1>CPSU</h1>

        <h2>
          Student Attendance Event<br />
          Monitoring System
        </h2>

        <div class="yellow-line"></div>

        

      </div>

      <div class="brand-footer">
        <span>Central Philippine State University</span>
      </div>

    </section>


    <!-- RIGHT LOGIN PANEL -->
    <section class="login-panel">

      <div class="login-card">

        <div class="mobile-logo">
          <div class="mobile-logo-circle">
            QR
          </div>
        </div>

        <div class="welcome">

          <span class="small-title">
            WELCOME TO CPSU
          </span>

          <h2>
            Attendance Portal
          </h2>

          <p>
            Sign in to continue to your account.
          </p>

        </div>


        <!-- LOGIN TYPE -->
        <div class="login-tabs">

          <button
            type="button"
            :class="{ active: loginType === 'student' }"
            @click="switchLoginType('student')"
          >
            Student
          </button>

          <button
            type="button"
            :class="{ active: loginType === 'admin' }"
            @click="switchLoginType('admin')"
          >
            Admin
          </button>

        </div>


        <!-- STUDENT LOGIN -->
        <form
          v-if="loginType === 'student'"
          @submit.prevent="login"
          class="login-form"
        >

          <div class="input-group">

            <label>Student ID</label>

            <div class="input-wrapper">

              <span class="input-icon">ID</span>

              <input
                v-model="studentId"
                type="text"
                placeholder="Enter your Student ID"
              />

            </div>

          </div>


          <div class="input-group">

            <label>Password</label>

            <div class="input-wrapper">

              <span class="input-icon">••</span>

              <input
                v-model="studentPassword"
                type="password"
                placeholder="Enter your password"
              />

            </div>

          </div>


          <div
            v-if="errorMessage"
            class="error-message"
          >
            {{ errorMessage }}
          </div>


          <button
            type="submit"
            class="login-button"
          >
            <span>Login</span>
            <span class="arrow">→</span>
          </button>


          <div class="register-area">

            <span>Don't have an account?</span>

            <button
              type="button"
              class="register-button"
              @click="emit('register')"
            >
              Create Student Account
            </button>

          </div>

        </form>


        <!-- ADMIN LOGIN -->
        <form
          v-else
          @submit.prevent="login"
          class="login-form"
        >

          <div class="input-group">

            <label>Username</label>

            <div class="input-wrapper">

              <span class="input-icon">@</span>

              <input
                v-model="username"
                type="text"
                placeholder="Enter admin username"
              />

            </div>

          </div>


          <div class="input-group">

            <label>Password</label>

            <div class="input-wrapper">

              <span class="input-icon">••</span>

              <input
                v-model="adminPassword"
                type="password"
                placeholder="Enter admin password"
              />

            </div>

          </div>


          <div
            v-if="errorMessage"
            class="error-message"
          >
            {{ errorMessage }}
          </div>


          <button
            type="submit"
            class="login-button"
          >
            <span>Login as Admin</span>
            <span class="arrow">→</span>
          </button>


          <div class="admin-note">
            <strong>Admin Access</strong>
            <span>
              Authorized administrators only
            </span>
          </div>

        </form>


        <div class="security-note">

          <span class="lock-icon">✓</span>

          <span>
            Your attendance information is securely managed.
          </span>

        </div>

      </div>

    </section>

  </div>
</template>


<style scoped>
* {
  box-sizing: border-box;
}

.login-page {
  min-height: 100vh;
  display: flex;
  background: #f0fdf4;
  font-family: Arial, Helvetica, sans-serif;
}


/* =========================
   LEFT BRAND PANEL
========================= */

.brand-panel {
  width: 48%;
  min-height: 100vh;
  background: #166534;
  color: white;
  position: relative;
  overflow: hidden;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 50px;
}

.brand-panel::before {
  content: "";
  position: absolute;
  width: 380px;
  height: 380px;
  border-radius: 50%;
  background: rgba(250, 204, 21, 0.08);
  top: -120px;
  left: -120px;
}

.brand-panel::after {
  content: "";
  position: absolute;
  width: 500px;
  height: 500px;
  border-radius: 50%;
  border: 2px solid rgba(250, 204, 21, 0.12);
  bottom: -260px;
  right: -180px;
}

.brand-content {
  position: relative;
  z-index: 2;
  max-width: 480px;
  text-align: center;
}


/* LOGO */

.school-logo {
  display: flex;
  justify-content: center;
  margin-bottom: 25px;
}

.logo-circle {
  width: 105px;
  height: 105px;
  border-radius: 50%;
  background: white;
  border: 7px solid #facc15;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #166534;
  font-size: 27px;
  font-weight: 800;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.18);
}

.brand-content h1 {
  color: #facc15;
  font-size: 52px;
  letter-spacing: 5px;
  margin-bottom: 8px;
}

.brand-content h2 {
  font-size: 25px;
  line-height: 1.35;
  font-weight: 700;
  margin-bottom: 25px;
}

.yellow-line {
  width: 70px;
  height: 5px;
  background: #facc15;
  border-radius: 20px;
  margin: 0 auto 25px;
}

.brand-content p {
  color: #dcfce7;
  font-size: 15px;
  line-height: 1.7;
  max-width: 390px;
  margin: auto;
}

.brand-footer {
  position: absolute;
  bottom: 30px;
  left: 0;
  width: 100%;
  text-align: center;
  color: #bbf7d0;
  font-size: 12px;
  z-index: 2;
}


/* =========================
   RIGHT LOGIN PANEL
========================= */

.login-panel {
  width: 52%;
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 40px;
  background: white;
}

.login-card {
  width: 100%;
  max-width: 470px;
}


/* WELCOME */

.welcome {
  margin-bottom: 25px;
}

.small-title {
  color: #15803d;
  font-size: 12px;
  font-weight: 800;
  letter-spacing: 1.5px;
}

.welcome h2 {
  color: #14532d;
  font-size: 32px;
  margin: 8px 0;
}

.welcome p {
  color: #6b7280;
  font-size: 14px;
}


/* MOBILE LOGO */

.mobile-logo {
  display: none;
}


/* =========================
   TABS
========================= */

.login-tabs {
  display: grid;
  grid-template-columns: 1fr 1fr;
  background: #f0fdf4;
  padding: 5px;
  border-radius: 12px;
  margin-bottom: 25px;
}

.login-tabs button {
  border: none;
  background: transparent;
  padding: 12px;
  border-radius: 9px;
  color: #6b7280;
  font-size: 14px;
  font-weight: 700;
  cursor: pointer;
  transition: 0.2s;
}

.login-tabs button.active {
  background: white;
  color: #166534;
  box-shadow: 0 2px 8px rgba(22, 101, 52, 0.12);
}

.login-tabs button:hover {
  color: #166534;
}


/* =========================
   FORM
========================= */

.login-form {
  display: flex;
  flex-direction: column;
  gap: 19px;
}

.input-group {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.input-group label {
  color: #374151;
  font-size: 13px;
  font-weight: 700;
}

.input-wrapper {
  position: relative;
}

.input-wrapper input {
  width: 100%;
  height: 50px;
  border: 1px solid #d1d5db;
  border-radius: 10px;
  padding: 0 15px 0 52px;
  outline: none;
  font-size: 14px;
  color: #1f2937;
  background: white;
  transition: 0.2s;
}

.input-wrapper input:focus {
  border-color: #22c55e;
  box-shadow: 0 0 0 3px rgba(34, 197, 94, 0.12);
}

.input-wrapper input::placeholder {
  color: #9ca3af;
}

.input-icon {
  position: absolute;
  left: 17px;
  top: 50%;
  transform: translateY(-50%);
  color: #166534;
  font-size: 12px;
  font-weight: 800;
  z-index: 1;
}


/* ERROR */

.error-message {
  background: #fef2f2;
  border: 1px solid #fecaca;
  color: #b91c1c;
  padding: 11px 13px;
  border-radius: 8px;
  font-size: 13px;
}


/* LOGIN BUTTON */

.login-button {
  width: 100%;
  height: 52px;
  border: none;
  border-radius: 10px;
  background: #166534;
  color: white;
  font-size: 15px;
  font-weight: 700;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 12px;
  transition: 0.2s;
  box-shadow: 0 5px 15px rgba(22, 101, 52, 0.18);
}

.login-button:hover {
  background: #14532d;
  transform: translateY(-1px);
}

.arrow {
  color: #facc15;
  font-size: 20px;
}


/* REGISTER */

.register-area {
  text-align: center;
  margin-top: 3px;
  font-size: 13px;
  color: #6b7280;
}

.register-button {
  border: none;
  background: transparent;
  color: #15803d;
  font-weight: 700;
  margin-left: 5px;
  cursor: pointer;
}

.register-button:hover {
  color: #ca8a04;
  text-decoration: underline;
}


/* ADMIN NOTE */

.admin-note {
  background: #fefce8;
  border: 1px solid #fde68a;
  border-radius: 10px;
  padding: 13px 15px;
  display: flex;
  flex-direction: column;
  gap: 3px;
  font-size: 12px;
}

.admin-note strong {
  color: #854d0e;
}

.admin-note span {
  color: #78716c;
}


/* SECURITY */

.security-note {
  margin-top: 28px;
  padding-top: 18px;
  border-top: 1px solid #e5e7eb;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  color: #6b7280;
  font-size: 11px;
  text-align: center;
}

.lock-icon {
  width: 20px;
  height: 20px;
  border-radius: 50%;
  background: #dcfce7;
  color: #15803d;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: bold;
}


/* =========================
   RESPONSIVE
========================= */

@media (max-width: 850px) {

  .brand-panel {
    width: 42%;
    padding: 25px;
  }

  .login-panel {
    width: 58%;
    padding: 30px;
  }

  .brand-content h1 {
    font-size: 42px;
  }

  .brand-content h2 {
    font-size: 21px;
  }
}


@media (max-width: 650px) {

  .login-page {
    display: block;
    background: #f0fdf4;
  }

  .brand-panel {
    width: 100%;
    min-height: 230px;
    padding: 30px 20px;
  }

  .brand-content {
    max-width: 400px;
  }

  .logo-circle {
    width: 70px;
    height: 70px;
    border-width: 5px;
    font-size: 19px;
  }

  .school-logo {
    margin-bottom: 12px;
  }

  .brand-content h1 {
    font-size: 32px;
  }

  .brand-content h2 {
    font-size: 17px;
    margin-bottom: 12px;
  }

  .brand-content p {
    display: none;
  }

  .yellow-line {
    width: 50px;
    height: 3px;
    margin-bottom: 0;
  }

  .brand-footer {
    display: none;
  }

  .login-panel {
    width: 100%;
    min-height: calc(100vh - 230px);
    padding: 30px 20px;
    align-items: flex-start;
  }

  .login-card {
    max-width: 500px;
  }

  .welcome h2 {
    font-size: 27px;
  }
}
</style>