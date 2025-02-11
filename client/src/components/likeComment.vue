<template>
    <div>
        <div>
            <button class="thumbs-up-comment" v-on:click="updateCommentLikes(passedComment.comment)">
                 <img style="width: 48%" src="/up.png"/> {{ passedComment.comment.likes }}
            </button>
            <button class="thumbs-down-comment" v-on:click="updateCommentDislikes(passedComment.comment)">
                 {{ passedComment.comment.dislikes }} <img style="width: 48%" src="/down.png"/>
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
      accountEmail: '', // Stores the logged-in user's email address
      commentLikes: 0, // Unused data property - consider removing
      commentDislikes: 0, // Unused data property - consider removing
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
      likedComments: [{ // Array of liked/disliked comments for the logged-in user
        comment: '',
        liked: false,
        disliked: false
      }]
    }
  },
  props: {
    passedComment: { // Receives comment data as a prop (likely from a parent component)
      type: Object, // Better to be explicit about the prop type
      required: false, //  And if it is required.
      default: () => ({}), // Provide a default value to avoid errors if the prop is not passed.
      // _id: String,  // While this is also possible, it's less flexible.
      // text: String,
      // likes: Number,
      // dislikes: Number,
      // name: String,
      // __v: Number
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
          this.likedNames = response.data.user.account.likedNames; // Store liked names.
          this.likedComments = response.data.user.account.likedComments; // Store liked/disliked comments
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

    // Updates comment likes and sends updates to the API
    updateCommentLikes(comment) {
      if (this.hasAccount) { // Only allow liking if the user is logged in
        let hasComment = false; // Flag to track if the comment is already in likedComments
        const upComment = { // Data to be sent in the PATCH request for the comment
          likes: comment.likes,
          dislikes: comment.dislikes
        };
        const likedComment = { // Data to be sent in the PATCH request for the user's likedComments
          comment: comment._id,
          liked: true,
          disliked: false
        };

        // Check if the comment is already in the user's likedComments
        for (let i = 0; i < this.likedComments.length; i++) {
          if (this.likedComments[i].comment === comment._id) {
            hasComment = true;
            if (!this.likedComments[i].liked) { // If not already liked, increment likes
              comment.likes += 1;
              upComment.likes += 1;
            } else { // If already liked, decrement likes
              comment.likes -= 1;
              upComment.likes -= 1;
            }
            this.likedComments[i].liked = !this.likedComments[i].liked; // Toggle the liked status
            likedComment.liked = this.likedComments[i].liked; // Update liked status for the PATCH request
          }
        }

        if (!hasComment) { // If the comment wasn't in likedComments, add it
          this.likedComments.push(likedComment);
          comment.likes += 1;
        }

        // Send PATCH requests to update the comment and the user's likedComments
        Api.patch('/v1/names/' + this.$route.params.id + '/comments/' + comment._id, upComment)
          .catch(error => {
            console.error(error); // Use console.error for errors
          });
        Api.patch('/v1/accounts/' + this.accountEmail + '/likedComments', likedComment)
          .catch(error => {
            console.error(error); // Use console.error for errors
          });
      }
    },

    // Updates comment dislikes (similar logic to updateCommentLikes)
    updateCommentDislikes(comment) {
      if (this.hasAccount) {
        let hasComment = false;
        const downComment = {
          likes: comment.likes,
          dislikes: comment.dislikes
        };
        const dislikedComment = {
          comment: comment._id,
          liked: false,
          disliked: true
        };

        for (let i = 0; i < this.likedComments.length; i++) {
          if (this.likedComments[i].comment === comment._id) {
            hasComment = true;
            if (!this.likedComments[i].disliked) {
              comment.dislikes += 1;
              downComment.dislikes += 1;
            } else {
              comment.dislikes -= 1;
              downComment.dislikes -= 1;
            }
            this.likedComments[i].disliked = !this.likedComments[i].disliked;
            dislikedComment.disliked = this.likedComments[i].disliked;
          }
        }

        if (!hasComment) {
          this.likedComments.push(dislikedComment);
          comment.dislikes += 1;
        }

        Api.patch('/v1/names/' + this.$route.params.id + '/comments/' + comment._id, downComment)
          .catch(error => {
            console.error(error);
          });
        Api.patch('/v1/accounts/' + this.accountEmail + '/likedComments', dislikedComment)
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
.thumbs-up-comment{
  width: 8%;
  height: 8%;
  border: 1px solid #74E3FC;
  border-radius: 30px 0px 0px 30px;
  background: linear-gradient(0deg, rgba(92, 93, 94, 0.2), rgba(92, 93, 94, 0.2)), #272727;
  color:#ffffff
}
.thumbs-down-comment{
  width: 8%;
  height: 8%;
  border: 1px solid #74E3FC;
  border-radius: 0px 30px 30px 0px;
  background: linear-gradient(0deg, rgba(92, 93, 94, 0.2), rgba(92, 93, 94, 0.2)), #272727;
  color:#ffffff
}
@media(max-width:768px){
  #box {
    padding: 20px;
  }
#likes-comment{
  margin-right: 4em;
}
.thumbs-up-comment{
  width: 50px;
  height: 30px;
}
.thumbs-down-comment{
  width: 50px;
  height: 30px;
}
}

</style>
