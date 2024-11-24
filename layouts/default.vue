<template>
  <v-app dark>
    <v-app-bar v-if="showNavbar" app fixed elevate height="80vh">
      <v-btn icon @click="drawer = !drawer" aria-label="Navigation Menu" v-if="showMenuIcon">
        <v-icon>mdi-menu</v-icon>
      </v-btn>

      <v-toolbar-title class="logo-title">
        <img src="@/assets/logo 3.png" alt="Victory One Logo" style="height: 80px;" />
      </v-toolbar-title>

      <v-spacer></v-spacer>

      <v-btn text :class="{ 'clicked': isHomeActive }" @click="goHome">
        HOME
      </v-btn>

      <v-btn text :class="{ 'clicked': isAboutUsActive }" to="/aboutus">
        ABOUT US
      </v-btn>

      <v-btn text :class="{ 'clicked': isContactUsActive }" to="/contactus">
        CONTACT US
      </v-btn>

      <v-menu offset-y>
        <template #activator="{ on, attrs }">
          <v-btn text v-bind="attrs" v-on="on">
            CATEGORIES <v-icon right>mdi-chevron-down</v-icon>
          </v-btn>
        </template>
        <v-list dense>
          <v-list-item v-for="(category, index) in categories" :key="index">
            <v-btn text @click="selectCategory(category)" class="category-button" style="width: 100%;">
              {{ category }}
            </v-btn>
          </v-list-item>
        </v-list>
      </v-menu>

      <v-spacer></v-spacer>

      <!-- Add to Cart Icon with Badge -->
      <v-btn icon to="/cart">
        <v-icon>mdi-cart-outline</v-icon>
        <v-badge v-if="cartItemCount > 0" color="red" :content="cartItemCount" overlap>
        </v-badge>
      </v-btn>

      <v-menu offset-y>
        <template #activator="{ on, attrs }">
          <v-avatar size="32" v-bind="attrs" v-on="on">
            <img
              :src="userProfilePic || 'https://static.vecteezy.com/system/resources/thumbnails/009/292/244/small_2x/default-avatar-icon-of-social-media-user-vector.jpg'"
              alt="User">
          </v-avatar>
        </template>

        <v-list>
          <v-list-item @click="ProfilePage">
            <v-list-item-title>Profile</v-list-item-title>
          </v-list-item>

          <v-list-item @click="openLogoutConfirmation" :disabled="!isUserLoggedIn"> <!-- Disable if not logged in -->
            <v-list-item-title>Logout</v-list-item-title>
          </v-list-item>
        </v-list>
      </v-menu>
    </v-app-bar>

    <!-- Confirmation Dialog for Logout -->
    <v-dialog v-model="logoutDialog" max-width="400">
      <v-card>
        <v-card-title class="headline">Confirm Logout</v-card-title>
        <v-card-text>
          Are you sure you want to log out?
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="primary" @click="logout">Yes</v-btn> <!-- Calls logout method -->
          <v-btn color="grey" @click="logoutDialog = false">No</v-btn> <!-- Closes dialog -->
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!-- Fullscreen Carousel -->
    <v-carousel v-if="showCarousel" height="50vh" hide-delimiter-background :cycle="true">
      <v-carousel-item v-for="(image, index) in images" :key="index">
        <v-img :src="image" class="fill-height">
          <v-container fluid class="d-flex align-center justify-center" style="height: 100%;">
            <v-row>
              <v-col class="text-center">
                <h2 class="carousel-text">{{ carouselTitles[index] }}</h2>
              </v-col>
            </v-row>
          </v-container>
        </v-img>
      </v-carousel-item>
    </v-carousel>

    <v-main>
      <v-container>
        <Nuxt />
      </v-container>
    </v-main>

    <v-footer class="footer" height="auto" v-if="showFooter">
      <v-container>
        <v-row>
          <!-- Stay Connected Section -->
          <v-col cols="12" md="4">
            <h3>Stay Connected</h3>
            <v-text-field label="Email Address" solo></v-text-field>
            <v-btn block style="margin-top: -20px; background-color: #FFA900; color: white;">SEND</v-btn>
            <div class="social-icons">
              <a href="https://www.facebook.com" target="_blank" rel="noopener noreferrer">
                <v-icon small>mdi-facebook</v-icon>
              </a>
              <a href="https://www.twitter.com" target="_blank" rel="noopener noreferrer">
                <v-icon small>mdi-twitter</v-icon>
              </a>
              <a href="https://www.google.com" target="_blank" rel="noopener noreferrer">
                <v-icon small>mdi-google-plus</v-icon>
              </a>
            </div>
          </v-col>

          <!-- Commitment Section -->
          <v-col cols="12" md="4">
            <h3>Victory One</h3>
            <p>We are proud to serve the community with dedication and care, ensuring a positive impact for future
              generations.</p>
          </v-col>

          <!-- Sitemap & Privacy Section -->
          <v-col cols="12" md="4">
            <h3 class="h3-1">Navigate</h3>
            <v-list style="background-color: #333;">
              <v-list-item>
                <v-btn text class="white-text" to="/">Home</v-btn>
              </v-list-item>
              <v-list-item>
                <v-btn text class="white-text" to="/aboutus">About us</v-btn>
              </v-list-item>
              <v-list-item>
                <v-btn text class="white-text" to="/contactus">Contact us</v-btn>
              </v-list-item>
              <v-list-item>
                <v-btn text class="white-text" to="/cart">My cart</v-btn>
              </v-list-item>
              <v-list-item>
                <v-btn text class="white-text" to="/privacy">Privacy Policy</v-btn>
              </v-list-item>
            </v-list>
          </v-col>
        </v-row>

        <v-row>
          <v-col class="text-center">
            <v-divider></v-divider>
            <p>&copy; 2024 Victory One. All rights reserved.</p>
          </v-col>
        </v-row>
      </v-container>
    </v-footer>
  </v-app>
</template>

<script>
import { firestore } from '~/plugins/firebase';
import { collection, getDocs, query, where, onSnapshot } from 'firebase/firestore';
import { getAuth, onAuthStateChanged, signOut } from 'firebase/auth';

export default {
  name: 'DefaultLayout',
  data() {
    return {
      categories: [],
      drawer: false,
      logoutDialog: false,
      images: [
        require('@/assets/carousel 3.jpg'),
        require('@/assets/carousel 1.jpeg'),
        require('@/assets/carousel 2.jpg'),
      ],
      carouselTitles: ['ALUMINUM', 'FRAMES', 'GLASS'],
      cartItemCount: 0,
      isUserLoggedIn: false,
      userProfilePic: '',
      userID: '',
    };
  },
  computed: {
    showNavbar() {
      return !['/sign/signin', '/sign/signup', '/sign/before', 'orderConfirmationSuccess'].includes(this.$route.path);
    },
    showFooter() {
      return ['/', '/aboutus', '/contactus', '/profile', '/cart', '/privacy', '/rating', '/support', '/gallery'].includes(this.$route.path);
    },
    showCarousel() {
      return ['/', '/gallery'].includes(this.$route.path);
    },
    showMenuIcon() {
      return ['/admin'].includes(this.$route.path);
    },
    isHomeActive() {
      return this.$route.path === '/';
    },
    isAboutUsActive() {
      return this.$route.path === '/aboutus';
    },
    isContactUsActive() {
      return this.$route.path === '/contactus';
    },
  },
  methods: {
    goHome() {
      this.$router.push('/');
    },
    ProfilePage() {
      this.$router.push('/profile');
    },
    async getCategories() {
      try {
        const querySnapshot = await getDocs(collection(firestore, 'Categories'));
        this.categories = querySnapshot.docs.map(doc => doc.data().ProductType); // Fetch the "ProductType" field
      } catch (error) {
        console.error("Error fetching categories:", error);
      }
    },
    selectCategory(category) {
      this.$router.push({ path: '/gallery', query: { category } });
    },
    openLogoutConfirmation() {
      this.logoutDialog = true;
    },
    logout() {
      const auth = getAuth();
      signOut(auth)
        .then(() => {
          this.isUserLoggedIn = false;
          this.userID = '';
          this.$router.push('/');
          this.logoutDialog = false;
        })
        .catch((error) => {
          console.error("Error logging out:", error);
        });
    },
    checkUserAuth() {
      const auth = getAuth();
      onAuthStateChanged(auth, (user) => {
        if (user) {
          this.isUserLoggedIn = true;
          this.userID = user.uid;
          this.getCartItems(); // Fetch cart items if user is logged in
        } else {
          this.isUserLoggedIn = false;
          this.userID = '';
          this.getCartItemsFromLocalStorage(); // Get cart items from localStorage if user is not logged in
        }
      });
    },
    getCartItems() {
      if (!this.isUserLoggedIn) return; // Only proceed if the user is logged in

      const cartRef = collection(firestore, 'Cart');
      const cartQuery = query(cartRef, where('userID', '==', this.userID));

      // Real-time listener for cart items
      onSnapshot(cartQuery, (querySnapshot) => {
        this.cartItemCount = querySnapshot.size; // Update cart item count dynamically
      });
    },
    getCartItemsFromLocalStorage() {
      if (process.client) {
        const cartItems = JSON.parse(localStorage.getItem('cartItems')) || [];
        this.cartItemCount = cartItems.length; // Update cart count from localStorage
      }
    },
  },
  created() {
    this.getCategories();
    this.checkUserAuth();
  },
};
</script>

<style scoped>
.footer {
  background-color: #333;
  color: #fff;
  padding: 20px 0;
}

.h3 {
  margin-left: 17px;
  color: white;
}

.h3-1 {
  margin-left: 16px;
}

.social-icons {
  display: flex;
  gap: 10px;
  margin-top: 10px;
}

.v-icon {
  color: white;
  /* Default color */
  transition: color 0.3s ease;
  /* Smooth transition for color change */
}

.social-icons a:hover .v-icon {
  color: #FFA900;
  /* Color change on hover */
}

.v-list-item {
  cursor: pointer;
}

.white-text {
  color: white;
}

.logo-title {
  display: flex;
  align-items: center;
}

.v-card {
  padding: 16px;
}

.v-btn {
  margin-right: 8px;
}

.clicked {
  background-color: rgba(0, 0, 0, 0.1);
  border-radius: 4px;
  transition: background-color 0.3s ease;
}

.admin-dashboard-title {
  color: white;
  font-weight: bold;
  font-size: 18px;
  padding: 16px;
  margin-bottom: 16px;
  text-align: center;
}

.drawer-item {
  font-size: 14px;
  color: black;
}

.carousel-text {
  font-size: 36px;
  font-family: 'Sequel Sans', sans-serif;
  text-align: center;
  position: relative;
  color: white;
}

.carousel-text::after {
  content: '';
  display: block;
  width: 50px;
  height: 4px;
  background-color: #FFA900;
  margin: 0 auto;
  margin-top: 10px;
}

.v-badge {
  position: relative;
  top: -10px;
  right: -10px;
}

.category-button {
  justify-content: flex-start;
  color: #3e3e3e;
  transition: background-color 0.3s ease;
}

.category-button:hover {
  background-color: rgba(0, 0, 0, 0.1);
}

.category-button .v-icon {
  margin-right: 8px;
}

.black-icon {
  color: white !important;
  transition: none;
}

.white-icon {
  color: white;
}
</style>