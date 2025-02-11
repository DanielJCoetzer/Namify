<template>
    <div>
      <div class="log-in-box">
      <form @submit.prevent="handleSubmit">
          <h1>Sign Up</h1>
        <div class="text_box">
          <input
            type="text"
            class="form-control"
            v-model="id"
            placeholder="Name"
          />
        </div>
        <div class="text_box">
          <input
            type="email"
            class="form-control"
            v-model="email"
            placeholder="Email"
          />
        </div>
        <div class="text_box">
          <input
            type="password"
            class="form-control"
            v-model="password"
            placeholder="Password"
          />
        </div>
        <button class="submit-button">Submit</button>
      </form>
      <p>Already have an account? <button @click="logIn()">Log In</button></p>
    </div>
    </div>
  </template>

<script>
import { Api } from '@/Api'

export default {
  name: 'createAccount', // Component name
  data() {
    return {
      id: '', // Stores the account name (ID) input value
      email: '', // Stores the email input value
      password: '', // Stores the password input value
      error: '' // Stores any error messages from the API
    }
  },
  methods: {
    // Emits a 'logIn' event, likely to trigger a login flow in the parent component
    logIn() {
      this.$emit('logIn')
    },

    // Handles form submission, including validation and API call to create a new account
    handleSubmit() {
      const newAccount = {
        name: this.id, // Use the entered ID as the account name
        email: this.email,
        password: this.password,
        likedNames: [], // Initialize empty likedNames array
        likedComments: [] // Initialize empty likedComments array
      }

      // Validate the form before making the API call
      if (this.validateForm()) {
        console.log('hello') // Placeholder log message

        // Make the API call to create the new account
        Api.post('/v1/accounts', newAccount)
          .then((res) => {
            console.log(res) // Log the successful response
            // Display a success message and potentially redirect (currently redirects to current page)
            this.$bvModal.msgBoxOk('SignUp Successful', this.$router.go(0))  //Consider changing this to a more appropriate route.
          }, (err) => {
            console.log(err) // Log the error response
            this.boxOne = '' // Reset a variable (purpose unclear, might be related to UI)
            this.error = err.response.data.error // Extract the error message from the API response
            // Display the error message in a modal
            this.$bvModal.msgBoxOk(this.error)
          })
      }
    },

    // Validates the create account form, checking for empty name and password fields
    validateForm() {
      const name = this.id // Use this.id, not this.name
      const password = this.password
      if (name === '' || password === '') {
        alert('Name and password cannot be empty') // Show an alert if either field is empty
        return false // Return false to indicate validation failure
      }
      return true // Return true if validation passes
    }
  }
}
</script>
  <style scoped>
  .head h1 {
    margin-top: 50px;
  }
  </style>
