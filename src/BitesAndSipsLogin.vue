<template>
  <div>
    <button v-if="token == null || token.length<1" @click="togglePopup">Login</button>
    <button v-else @click="logout">Logout</button>
    <div class="popup" v-if="showPopup">
      <div class="popup-inner">
        <h2>Login</h2>
        <form>
          <label for="username">Email:</label>
          <input type="text" id="username" v-model="username">
          <label for="password">Password:</label>
          <input type="password" id="password" v-model="password">
          <button type="button" @click="login">Sign In</button>
        </form>
        <button class="close" @click="closePopup">X</button>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: 'BitesAndSipsLogin',
  props: {
    token: String
  },
  data() {
    return {
      showPopup: false,
      username: '',
      password: ''
    }
  },
  methods: {
    togglePopup() {
      this.showPopup = !this.showPopup;
    },
    closePopup() {
      this.showPopup = false;
    },
    login() {
      // Hier würde normalerweise die Logik für die Authentifizierung stattfinden
      axios.post('https://localhost/api/login', { "email": this.username, "password": this.password })
        .then(response => {
          this.$emit('set-token', response.data.access_token)
        })
        .catch(() => {
          // Handle login error
          this.$emit('setErrorText', 'An error has occurred');
        });
      this.closePopup();
    },
    logout(){
      axios.post('https://localhost/api/logout')
        .then(() => {
          localStorage.removeItem('access_token')
          axios.defaults.headers.common['Authorization']
          // Redirect to protected route
        })
        .catch(() => {
          // Handle login error
          this.$emit('setErrorText', 'An error has occurred');
        });
    },
  }
}
</script>

<style>
.popup {
  position: fixed;
  z-index: 1;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  overflow: auto;
  background-color: rgba(0, 0, 0, 0.4);
  display: flex;
  justify-content: center;
  align-items: center;
}

.popup-inner {
  width: 50%;
  min-width: 300px;
  max-width: 500px;
  background-color: #fefefe;
  border-radius: 5px;
  padding: 20px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.3);
  position: relative;
}

.popup-inner h2 {
  margin-top: 0;
}

.popup-inner label {
  display: block;
  margin-bottom: 10px;
}

.popup-inner input {
  display: block;
  margin-bottom: 20px;
  width: 100%;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 3px;
  box-sizing: border-box;
}

.popup-inner button[type="button"] {
  background-color: #0468eb;
  color: #fff;
  padding: 10px 20px;
  border: none;
  border-radius: 3px;
  cursor: pointer;
  text-decoration: underline;
}

.popup-inner .close {
  position: absolute;
  top: 10px;
  right: 10px;
  font-size: 20px;
  font-weight: bold;
  color: #aaa;
  cursor: pointer;
  border: none;
  background: none;
}
</style>
