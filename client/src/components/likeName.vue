<template>
    <div>
        <div>
        <button class="thumbs-up" v-on:click="updateLikes()">
            <img style="width: 30%" src="/up.png"/> {{this.name.likes}}
        </button>
        <button class="thumbs-down" v-on:click="updateDislikes()">
            {{this.name.dislikes}} <img style="width: 30%" src="/down.png"/>
        </button>
    </div>
    </div>
</template>

<script>
import { Api } from '@/Api'

export default {
  data() {
    return {
      hasAccount: false, // Tracks whether the user is logged in
      accountEmail: '', // Stores the logged-in user's email
      name: { // Stores the name data fetched from the API
        comments: [{ // Array of comments for the name
          _id: '',
          text: '',
          likes: 0,
          dislikes: 0,
          name: '',
          __v: 0
        }],
        tags: [], // Array of tags associated with the name
        _id: '',
        likes: 0,
        dislikes: 0,
        __v: 0
      },
      likedNames: [{ // Array of liked/disliked names for the logged-in user
        name: '',
        liked: false,
        disliked: false
      }]
    }
  },
  mounted() {
    this.getAccount(); // Fetch account details when the component is mounted
    this.getName(); // Fetch name details when the component is mounted
  },
  methods: {
    // Fetches account details from the API
    getAccount() {
      Api.get('/v1/accounts', { headers: { token: localStorage.getItem('token') } })
        .then(response => {
          this.hasAccount = true; // Set hasAccount to true if the API call is successful
          this.accountEmail = response.data.user.account.email; // Store the user's email
          this.likedNames = response.data.user.account.likedNames; // Store liked names
          this.likedComments = response.data.user.account.likedComments; // Store liked comments
        })
        .catch(err => {
          console.error(err.message + ', user is not logged in to an account'); // Log error if not logged in. Use console.error for errors.
          this.hasAccount = false; // Set hasAccount to false if the API call fails
        });
    },

    // Fetches name details from the API
    getName() {
      Api.get('/v1/names/' + this.$route.params.id)
        .then(response => {
          this.name = response.data; // Store the fetched name data
        });
    },

    // Updates name likes and sends updates to the API
    updateLikes() {
      if (this.hasAccount) { // Only allow liking if the user is logged in
        let hasName = false; // Flag to track if the name is already in likedNames
        const upName = { // Data to be sent in the PATCH request for the name
          likes: this.name.likes,
          dislikes: this.name.dislikes
        };
        const likedName = { // Data to be sent in the PATCH request for the user's likedNames
          name: this.name._id,
          liked: true,
          disliked: false
        };

        // Check if the name is already in the user's likedNames
        for (let i = 0; i < this.likedNames.length; i++) {
          if (this.likedNames[i].name === this.name._id) {
            hasName = true;
            if (!this.likedNames[i].liked) { // If not already liked, increment likes
              this.name.likes += 1;
              upName.likes += 1;
            } else { // If already liked, decrement likes
              this.name.likes -= 1;
              upName.likes -= 1;
            }
            this.likedNames[i].liked = !this.likedNames[i].liked; // Toggle the liked status
            likedName.liked = this.likedNames[i].liked; // Update liked status for the PATCH request
          }
        }

        if (!hasName) { // If the name wasn't in likedNames, add it
          this.likedNames.push(likedName);
          this.name.likes += 1;
        }

        // Send PATCH requests to update the name and the user's likedNames
        Api.patch('/v1/names/' + this.$route.params.id, upName)
          .catch(error => {
            console.error(error); // Use console.error for errors
          });
        Api.patch('/v1/accounts/' + this.accountEmail + '/likedNames', likedName)
          .catch(error => {
            console.error(error); // Use console.error for errors
          });
      }
    },

    // Updates name dislikes (similar logic to updateLikes)
    updateDislikes() {
      if (this.hasAccount) {
        let hasName = false;
        const downName = {
          likes: this.name.likes,
          dislikes: this.name.dislikes
        };
        const dislikedName = {
          name: this.name._id,
          liked: false,
          disliked: true
        };

        for (let i = 0; i < this.likedNames.length; i++) {
          if (this.likedNames[i].name === this.name._id) {
            hasName = true;
            if (!this.likedNames[i].disliked) {
              this.name.dislikes += 1;
              downName.dislikes += 1;
            } else {
              this.name.dislikes -= 1;
              downName.dislikes -= 1;
            }
            this.likedNames[i].disliked = !this.likedNames[i].disliked;
            dislikedName.disliked = this.likedNames[i].disliked;
          }
        }

        if (!hasName) {
          this.likedNames.push(dislikedName);
          this.name.dislikes += 1;
        }

        Api.patch('/v1/names/' + this.$route.params.id, downName)
          .catch(error => {
            console.error(error);
          });
        Api.patch('/v1/accounts/' + this.accountEmail + '/likedNames', dislikedName)
          .catch(error => {
            console.error(error);
          });
      }
    }
  }
}
</script>

<style>
#likes {
    color: #FFFFFF;
    font-size: 20px;
    display:inline;
    vertical-align: top;
    text-align: right;
}
.thumbs-up{
  width: 36%;
  border: 1px solid #74E3FC;
  border-radius: 30px 0px 0px 30px;
  background: linear-gradient(0deg, rgba(92, 93, 94, 0.2), rgba(92, 93, 94, 0.2)), #272727;
  color:#ffffff
}
.thumbs-down{
  width: 36%;
  border: 1px solid #74E3FC;
  border-radius: 0px 30px 30px 0px;
  background: linear-gradient(0deg, rgba(92, 93, 94, 0.2), rgba(92, 93, 94, 0.2)), #272727;
  color:#ffffff
}
@media(max-width:768px){
  #box {
    padding: 20px;
  }
.thumbs-up{
  width: 80px;
  font-size: 15px;
  height: 40px;
  border-radius: 30px 0px 0px 30px;
  background: linear-gradient(0deg, rgba(92, 93, 94, 0.2), rgba(92, 93, 94, 0.2)), #272727;
}
.thumbs-down{
  font-size: 15px;
  width: 80px;
  height: 40px;
  border-radius: 0px 30px 30px 0px;
  background: linear-gradient(0deg, rgba(92, 93, 94, 0.2), rgba(92, 93, 94, 0.2)), #272727;
}
#likes{
  margin-right: 4.6em;
}
}
</style>
