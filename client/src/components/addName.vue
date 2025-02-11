<template>
    <div>
      <b-button class="name-button"  v-b-modal.modal-prevent-closing>add name</b-button>
      <div class="mt-3">
      </div>
      <b-modal
        id="modal-prevent-closing"
        ref="modal"
        title="Submit Your Name"
        @show="resetModal"
        @hidden="resetModal"
        @ok="handleOk"
      >
        <form ref="form" @submit.stop.prevent="handleSubmit">
          <b-form-group
            label="Name"
            label-for="name-input"
            invalid-feedback="Name is required"
            :state="nameState"
          >
            <b-form-input
              id="name-input"
              v-model="name"
              :state="nameState"
              required
            ></b-form-input>
          </b-form-group>
        </form>
      </b-modal>
    </div>
  </template>
<script>
import { Api } from '@/Api'

export default {
  data() {
    return {
      name: '', // Stores the name input value
      nameState: null, // Stores the validity state of the name input (true/false/null)
      submittedNames: [] // Array to store submitted names (currently unused)
    }
  },
  methods: {
    // Creates a new name by sending a POST request to the API
    createName() {
      const newName = {
        _id: this.name, // The entered name is used as the ID
        likes: 0,
        dislikes: 0,
        comments: [],
        tags: []
      }
      Api.post('v1/names', newName)
        .catch(error => {
          console.log(error) // Log any errors during API call
        })
      this.$router.push('/name/' + this.name) // Redirect to the newly created name's page
    },

    // Checks the validity of the form using Bootstrap Vue's form validation
    checkFormValidity() {
      const valid = this.$refs.form.checkValidity() // Get the validation result
      this.nameState = valid // Update the nameState
      return valid // Return the validation result
    },

    // Resets the modal form fields
    resetModal() {
      this.name = '' // Clear the name input
      this.nameState = null // Reset the name state
    },

    // Handles the "OK" button click in the modal, preventing default close and triggering submit
    handleOk(bvModalEvent) {
      bvModalEvent.preventDefault() // Prevent the modal from closing immediately
      this.handleSubmit() // Call the submit handler
    },

    // Handles form submission, including validation and API call
    handleSubmit() {
      if (!this.checkFormValidity()) { // Check form validity before submission
        return // Exit if the form is not valid
      }

      this.createName() // Create the name if the form is valid

      // Hide the modal manually after successful submission
      this.$nextTick(() => {
        this.$bvModal.hide('modal-prevent-closing') // Hide the modal using its ID
      })
    }
  }
}
</script>
<style>
  @media(max-width: 768px){
    .name-button{
  width: 100px;
  height: 40px;
  padding: 1px;
  margin-left: -28px;
  margin-top: -4px;
  margin-bottom: 4px;
  font-size: 5px;
  text-align: center;
    }
  }
  </style>
