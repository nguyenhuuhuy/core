<template>
  <form @submit.prevent="login" :class="{ error: failed }" data-cy="loginForm">
    <div class="logo">
      <img src="~#/../img/logo.svg" width="156" height="auto">
    </div>
    <input v-if="isDesktopApp" v-model="url" type="url" placeholder="Koel's Host" autofocus required>
    <input v-model="email" type="email" placeholder="Email Address" autofocus required>
    <input v-model="password" type="password" placeholder="Password" required>
    <button type="submit">Log In</button>
  </form>
</template>

<script>
import axios from 'axios'
import { userStore } from '@/stores'
import { ls } from '@/services'

const DEMO_ACCOUNT = {
  email: 'demo@koel.phanan.net',
  password: 'demo'
}

export default {
  data () {
    return {
      url: '',
      email: NODE_ENV === 'demo' ? DEMO_ACCOUNT.email : '',
      password: NODE_ENV === 'demo' ? DEMO_ACCOUNT.password : '',
      failed: false,
      isDesktopApp: KOEL_ENV === 'app'
    }
  },

  methods: {
    async login () {
      if (this.isDesktopApp) {
        axios.defaults.baseURL = (this.url.endsWith('/') ? this.url : `${this.url}/`) + 'api'
      }

      try {
        await userStore.login(this.email, this.password)
        this.failed = false

        // Reset the password so that the next login will have this field empty.
        this.password = ''

        if (this.isDesktopApp) {
          ls.set('koelHost', this.url.endsWith('/') ? this.url : `${this.url}/`)
          ls.set('lastLoginEmail', this.email)
        }

        this.$emit('loggedin')
      } catch (err) {
        this.failed = true
      }
    }
  },

  mounted () {
    if (!this.isDesktopApp) {
      return
    }

    this.url = window.BASE_URL = ls.get('koelHost')
    this.email = ls.get('lastLoginEmail')
  }
}
</script>

<style lang="scss" scoped>
/**
 * I like to move it move it
 * I like to move it move it
 * I like to move it move it
 * You like to - move it!
 */
@keyframes shake {
  8%, 41% {
    -webkit-transform: translateX(-10px);
  }
  25%, 58% {
    -webkit-transform: translateX(10px);
  }
  75% {
    -webkit-transform: translateX(-5px);
  }
  92% {
    -webkit-transform: translateX(5px);
  }
  0%, 100% {
    -webkit-transform: translateX(0);
  }
}

form {
  width: 280px;
  padding: 1.8rem;
  background: rgba(255,255,255,.08);
  border-radius: .6rem;
  border: 1px solid #333;

  &.error {
    border-color: #8e4947;
    animation: shake .5s;
  }

  .logo {
    text-align: center;
  }

  @media only screen and (max-width : 414px) {
    border: 0;
    background: transparent;
  }
}

input {
  display: block;
  margin-top: 12px;
  border: 0;
  background: #fff;
  outline: none;
  width: 100%;
}

button {
  display: block;
  margin-top: 12px;
  width: 100%;
}
</style>
