<template>
    <div>
      <input v-model="searchQuery">
      <router-link :to="{ name: 'name', params: { id: searchQuery } }">
      <!--<div v-for="name of resultQuery" :key="name._id">{{name._id}}</div> -->
    </router-link>
  </div>
</template>
<script>
import { Api } from '@/Api'

export default {
  data() {
    return {
      searchQuery: null, // Stores the user's search input
      names: { // Stores the names fetched from the API
        names: {} // Nested object to hold the array of names (structure from API response)
      },
      value: null // Unused data property
    }
  },
  computed: {
    // Computes the filtered list of names based on the search query
    resultQuery() {
      if (this.searchQuery) { // Check if there's a search query
        return this.names.names.filter(item => { // Filter the names array
          return this.searchQuery // The original filter condition was always true.
            .toLowerCase()
            .split(' ') // Split the search query into individual words
            .every(v => item.toLowerCase().includes(v)) // Check if every word in the query is present in the name
        })
      } else {
        return this.names.names // Return the original list if there's no search query
      }
    }
  },
  mounted() {
    console.log('Page is loaded!')

    // Fetch the list of names from the API when the component is mounted
    Api.get('/v1/names')
      .then((response) => {
        console.log(response) // Log the API response
        this.names = response.data // Assign the fetched data to the names object
      })
      .catch((error) => {
        this.names = [] // Set names to an empty array in case of an error
        console.log(error) // Log the error
      })
  }
}
</script>
<style>
.names h2 {
  background: #f4f4f4;
  padding: 20px;
  border-radius: 10px;
  margin: 10px 0;
  max-width: 600px;
  cursor: pointer;
  color: #444;
 }
.names h2:hover {
  background: #ddd;
 }
.names a {
  text-decoration: none;
 }
.names {
  max-width: 75vh;
  font-size: 1.5em;
  text-align: center;
}
.card-names {
  box-shadow: 0 1px 1px 0 rgba(255, 255, 255, 0.178);
  padding: 4%;
  border: 2px solid rgb(140, 231, 245);
  color: black;
  margin: 20px;
  background-color: rgba(1, 1, 1, 0.15);
}
.card-body-list {
  color: white;
  margin-left: 20%;
  text-align: left;
}
</style>
