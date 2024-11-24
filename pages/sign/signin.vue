<template>
  <v-container class="fill-height d-flex align-center justify-center" fluid>
    <v-row align="center" justify="center">
      <v-col cols="12" sm="8" md="5">
        <v-card class="sign-in-card">
          <v-card-text class="text-center mt-4">
            <h2>Sign In</h2>
          </v-card-text>

          <div class="text-center divider mt-2 mb-2">
            -----sign in using-----
          </div>

          <v-card-text>
            <v-row justify="center" class="social-login">
              <v-col cols="6">
                <v-btn @click="signInWithFacebook" block color="#1877F2" class="social-btn">
                  <v-icon left color="white">mdi-facebook</v-icon> Facebook
                </v-btn>
              </v-col>
              <v-col cols="6">
                <v-btn @click="signInWithGoogle" block color="#DB4437" class="social-btn">
                  <v-icon left color="white">mdi-google</v-icon> Google
                </v-btn>
              </v-col>
            </v-row>
          </v-card-text>

          <div class="text-center divider mt-2 mb-2">
            -----or-----
          </div>

          <v-card-text>
            <v-form ref="form" v-model="valid" lazy-validation>
              <v-text-field v-model="email" :rules="emailRules" label="Email" required class="input-field mt-2">
                <template v-slot:prepend>
                  <v-icon>mdi-email</v-icon>
                </template>
              </v-text-field>

              <v-text-field v-model="password" :rules="passwordRules" label="Password"
                :type="showPassword ? 'text' : 'password'" required class="input-field mt-2">
                <template v-slot:prepend>
                  <v-icon>mdi-lock</v-icon>
                </template>
                <template v-slot:append>
                  <v-btn icon @click="showPassword = !showPassword">
                    <v-icon>{{ showPassword ? 'mdi-eye-off' : 'mdi-eye' }}</v-icon>
                  </v-btn>
                </template>
              </v-text-field>

              <div class="text-center mb-1">
                <span class="character-info">8-25 Characters</span>
              </div>

              <v-btn @click="signIn" :disabled="loading" class="primary-btn full-width" block
                style="background-color: #000; color: #fff; font-weight: bold;">
                <v-icon v-if="loading" left>mdi-loading</v-icon>
                <span v-if="!loading">SIGN IN</span>
                <span v-else>Signing In...</span>
              </v-btn>
            </v-form>
          </v-card-text>

          <v-btn @click="goToSignUp" class="outlined-btn full-width" block
            style="border: 2px solid black; color: black; font-weight: bold;">
            CREATE ACCOUNT
          </v-btn>

          <div class="text-center mt-1">
            <v-btn text @click="forgotPasswordDialog = true" class="text-link" style="padding: 0; margin: 0;">
              Forgot Password?
            </v-btn>
          </div>

          <div class="text-center mt-2">
            <p class="additional-info">
              Track your order, save your favorites, get product recommendations & more!
            </p>
          </div>
        </v-card>
      </v-col>
    </v-row>

    <!-- Forgot Password Dialog -->
    <v-dialog v-model="forgotPasswordDialog" max-width="400px">
      <v-card>
        <v-card-title class="headline">Reset Password</v-card-title>
        <v-card-text>
          <p>Please enter your email address to reset your password:</p>
          <v-text-field v-model="resetEmail" :rules="emailRules" label="Email" required>
            <template v-slot:prepend>
              <v-icon>mdi-email</v-icon>
            </template>
          </v-text-field>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn style="background-color: #ffa900; color: white" @click="sendResetEmail">Send</v-btn>
          <v-btn text @click="forgotPasswordDialog = false">Cancel</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-container>
</template>

<script>
import { auth, firestore, googleProvider } from '~/plugins/firebase';
import { signInWithPopup, signInWithEmailAndPassword, sendPasswordResetEmail, createUserWithEmailAndPassword } from 'firebase/auth';
import { doc, getDoc, setDoc } from 'firebase/firestore';

export default {
  data() {
    return {
      loading: false,
      valid: false,
      email: '',
      password: '',
      showPassword: false,
      forgotPasswordDialog: false,  // This should be declared here
      resetEmail: '',
      showForgotPasswordDialog: false,  // <-- Add this line
      emailRules: [v => !!v || 'Email is required', v => /.+@.+\..+/.test(v) || 'E-mail must be valid'],
      passwordRules: [v => !!v || 'Password is required', v => v.length >= 6 || 'Password must be at least 6 characters'],
    };
  },
  methods: {
    async signIn() {
      if (this.$refs.form.validate()) {
        this.loading = true; // Start loading indicator
        try {
          const userCredential = await signInWithEmailAndPassword(auth, this.email, this.password);
          const userId = userCredential.user.uid;

          // Fetch the user's role from Firestore
          const userDocRef = doc(firestore, 'Users', userId);
          const userDoc = await getDoc(userDocRef);
          if (userDoc.exists()) {
            const userRole = userDoc.data().role;

            // Ensure the userID field is added to the user document
            await setDoc(userDocRef, { userID: userId }, { merge: true });

            // Check if there are any cart items in localStorage
            const cartItems = JSON.parse(localStorage.getItem('cart')) || [];
            if (cartItems.length > 0) {
              // Transfer items from localStorage to Firestore Cart collection
              const cartRef = collection(firestore, 'Cart');
              for (const item of cartItems) {
                await addDoc(cartRef, {
                  ProductID: item.ProductID,
                  Quantity: item.Quantity,
                  userID: userId,
                });
              }
              // Clear localStorage after transferring to Firestore
              localStorage.removeItem('cart');

              // Redirect to the checkout page
              this.$router.push('/checkout');
            } else {
              // Redirect to the home page if cart is empty
              this.$router.push('/');
            }
          } else {
            console.error("User role not found in Firestore");
          }
        } catch (error) {
          console.error("Error signing in:", error);
          alert("Sign-in failed: " + error.message);
        } finally {
          this.loading = false; // Stop loading indicator
        }
      }
    },
    async signInWithGoogle() {
      try {
        const result = await signInWithPopup(auth, googleProvider);
        const userId = result.user.uid;

        // Fetch the user's document from Firestore
        const userDocRef = doc(firestore, 'Users', userId);
        const userDoc = await getDoc(userDocRef);

        if (userDoc.exists()) {
          const cartItems = JSON.parse(localStorage.getItem('cart')) || [];
          if (cartItems.length > 0) {
            const cartRef = collection(firestore, 'Cart');
            for (const item of cartItems) {
              await addDoc(cartRef, {
                ProductID: item.ProductID,
                Quantity: item.Quantity,
                userID: userId,
              });
            }
            localStorage.removeItem('cart');
            this.$router.push('/checkout');
          } else {
            this.$router.push('/');
          }
        } else {
          // If the user does not exist, create the user document with a default role
          await setDoc(userDocRef, { userID: userId, role: 'customer' });
          this.$router.push('/');
        }
      } catch (error) {
        console.error("Google Sign-In Error:", error);
        alert("Google sign-in failed: " + error.message);
      }
    },
    async signInWithFacebook() {
      try {
        // Assuming you have a Facebook Auth provider set up
        const facebookProvider = new firebase.auth.FacebookAuthProvider();
        const result = await signInWithPopup(auth, facebookProvider);
        const userId = result.user.uid;

        // Fetch the user's role from Firestore
        const userDocRef = doc(firestore, 'Users', userId);
        const userDoc = await getDoc(userDocRef);
        if (userDoc.exists()) {
          const userRole = userDoc.data().role;

          // Ensure the userID field is added to the user document
          await setDoc(userDocRef, {
            userID: userId, // Add userID to the Firestore user document
          }, { merge: true });

          // Check if the user has an allowed role for protected routes
          if (['admin', 'cashier', 'owner'].includes(userRole)) {
            const items = this.$route.query.items || '[]'; // Get items from query
            this.$router.push({ path: '/checkout', query: { items } });
          } else {
            this.$router.push('/no-access');
          }
        } else {
          console.error("User role not found in Firestore");
        }
      } catch (error) {
        console.error("Facebook Sign-In Error:", error);
        alert("Facebook sign-in failed: " + error.message);
      }
    },
    async sendResetEmail() {
      if (this.resetEmail) {
        try {
          await sendPasswordResetEmail(auth, this.resetEmail);
          this.forgotPasswordDialog = false;
          alert("Password reset email sent!");
        } catch (error) {
          console.error("Error sending password reset email:", error);
          alert("Error sending password reset email: " + error.message);
        }
      } else {
        alert("Please enter your email to reset your password.");
      }
    },
    goToSignUp() {
      this.$router.push('/sign/signup');
    },
  },
};
</script>

<style scoped>
/* Add your styles here */
.fill-height {
  height: 100vh;
  background-color: white;
}

.sign-in-card {
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

.primary-btn {
  font-weight: bold;
  border-radius: 24px;
  height: 48px;
}

.outlined-btn {
  font-weight: bold;
  border-radius: 24px;
  height: 48px;
}

.outlined-btn1 {
  font-weight: bold;
  border-radius: 24px;
  height: 48px;
  margin-bottom: 10px;
  margin-top: -10px;
}

.social-btn {
  color: white;
  font-size: 16px;
  border-radius: 24px;
  height: 48px;
}

.divider {
  color: gray;
  font-size: 14px;
}

.text-link {
  font-size: 14px;
  color: #007bff;
  text-decoration: underline;
  cursor: pointer;
}

.character-info {
  font-size: 12px;
  color: gray;
}

.additional-info {
  font-size: 12px;
  color: gray;
}

.input-field {
  margin-top: 10px;
}

.text-center {
  margin-top: 2px;
}

.text-center h2 {
  font-size: 32px;
  color: black;
}
</style>