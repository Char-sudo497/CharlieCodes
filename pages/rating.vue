<template>
    <v-container>
      <v-row justify="center">
        <v-col cols="12" md="8">
          <v-card>
            <v-card-text class="text-center">
              <h2 style="font-weight: bold; color: black;">Rate Your Experience</h2>
              <p>We value your feedback! Please rate your experience and provide any comments below:</p>
              <v-rating
                v-model="rating"
                background-color="black"
                color="yellow darken-2"
                large
                empty-icon="mdi-star-outline"
                full-icon="mdi-star"
              ></v-rating>
              <v-form @submit.prevent="submitFeedback">
                <v-textarea
                  v-model="feedback"
                  label="Your Feedback"
                  rows="4"
                  outlined
                  class="mt-4"
                ></v-textarea>
                <v-text-field
                  v-model="email"
                  label="Your Email (optional)"
                  outlined
                  class="mt-4"
                ></v-text-field>
                <v-btn class="mt-4" type="submit" style="background-color: #ffa900; color: white;" block>Submit</v-btn>
              </v-form>
            </v-card-text>
          </v-card>
        </v-col>
      </v-row>
    </v-container>
  </template>
  
  <script>
  export default {
    data() {
      return {
        rating: 0,
        feedback: '',
        email: '',
      };
    },
    methods: {
      submitFeedback() {
        console.log('Rating:', this.rating);
        console.log('Feedback:', this.feedback);
        console.log('Email:', this.email);
        this.resetForm();
        this.$toast.success('Thank you for your feedback!');
      },
      resetForm() {
        this.rating = 0;
        this.feedback = '';
        this.email = '';
      },
    },
  };
  </script>
  
  <style scoped>
  .v-card {
    padding: 16px;
    max-width: 600px;
    margin: 0 auto;
    text-align: center;
  }
  
  h2 {
    margin-bottom: 5px;
  }
  
  .v-rating {
    margin-bottom: 16px;
  }
  </style>