

<template>
  <div>
    <h2>VUE 3 Google Login Demo</h2>

    <div v-show="!isLoggedIn" id="google-button"></div>
    <div v-if="isLoggedIn">
      <p>Welcome! {{email}} </p>
      <button @click="googleSignOut">
        Sign Out
      </button>
    </div>
    

  </div>
</template>

<script>
  import axios from 'axios';

  export default {
    data() {
      return {
        email: '',
        isLoggedIn: false,
      };
    },
    methods: {
      handleCredentialResponse(response) {
        const { credential } = response;
        axios({
          method: "POST",
          url: `http://localhost:3000/google-login`,
          data: {
            credential
          }
        })
        .then(({ data }) => {
          this.setLocalStorage(data);
          this.email = data.email
          this.isLoggedIn = true
        })
        .catch(err => {
          console.error("error", "Oops Login with Google failed...", err)
        })
      },
      setLocalStorage(data) {
        localStorage.access_token = data.accessToken;
        localStorage.email = data.email; 
      },

      googleSignOut(){
          google.accounts.id.disableAutoSelect();

          // REVOKE one tap access
          // google.accounts.id.revoke(localStorage.email, done => {
          //   console.log('consent revoked');
          // });

          this.resetState()
          localStorage.clear()
      },

      googleSignInOnLoad() {
        const cb = this.handleCredentialResponse

        window.onload = function() { 
          google.accounts.id.initialize({
            client_id: "225290592554-sk885pi38frutehh9g561mtdhpvg52mf.apps.googleusercontent.com",
            // auto_select: true, // optional
            callback: cb,
          });
          google.accounts.id.renderButton(
            document.getElementById("google-button"),
            { theme: "outline", size: "large" }  // customization attributes
          );

          google.accounts.id.prompt(); // also display the One Tap dialog
        }
      },

      resetState() {
        this.email = ''
        this.isLoggedIn = false
      }
    },

    mounted() {
      if (localStorage.email && localStorage.access_token) {
        this.email = localStorage.email
        this.isLoggedIn = true
      }

      this.googleSignInOnLoad()
    },
  };
</script>
