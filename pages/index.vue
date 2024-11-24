<template>
  <v-app>
    <v-container>
      <!-- MAIN PRODUCTS Header -->
      <v-row class="mb-4">
        <v-col cols="12">
          <h2 class="header-title">MAIN PRODUCTS</h2>
        </v-col>
      </v-row>

      <!-- Oval Button Styled List -->
      <v-row class="mb-4 justify-center">
        <v-col cols="12" class="text-center">
          <v-btn v-for="item in itemList" :key="item.id" class="mr-3 mb-2 oval-button" @click="handleItemClick(item)">
            {{ item.ProductType }}
          </v-btn>
        </v-col>
      </v-row>

      <!-- Product Grid -->
      <v-row>
        <v-col v-for="product in products" :key="product.id" cols="12" md="4">
          <v-card class="hover-card" @click="goToProduct(product.id)">
            <v-img :src="product.image" height="200px"></v-img>

            <v-card-title class="d-flex justify-space-between align-center">
              <span>{{ product.name }}</span>
              <v-btn icon class="cart-icon" @click.stop="addToCart(product)">
                <v-icon style="color: #FFA900;">mdi-cart-minus</v-icon>
              </v-btn>
            </v-card-title>

            <v-card-subtitle style="color: black;">₱{{ product.price }}</v-card-subtitle>

            <!-- Sold Quantity Display -->
            <v-card-subtitle class="sold-info" style="color: black;">Sold: {{ product.soldQuantity }}</v-card-subtitle>

            <!-- See More and Buy Now buttons -->
            <v-card-actions class="d-flex justify-space-between">
              <span class="see-more" @click.stop="goToProduct(product.id)">See More..</span>
              <v-btn class="buy-now-btn" @click.stop="buyNow(product)" style="background-color: #FFA900; color: white;">Buy Now</v-btn>
            </v-card-actions>
          </v-card>
        </v-col>
      </v-row>

      <!-- SEE MORE PRODUCTS Button -->
      <v-row class="mt-4 justify-center">
        <v-col cols="12" class="text-center">
          <v-btn class="see-more-btn" @click="goToGallery" style="background-color: #FFA900;">
            SEE MORE PRODUCTS <v-icon right>mdi-arrow-right</v-icon>
          </v-btn>
        </v-col>
      </v-row>
    </v-container>
  </v-app>
</template>

<script>
import { firestore } from '~/plugins/firebase';
import { collection, getDocs, addDoc, query, where, updateDoc } from 'firebase/firestore'; // Import query and where
import { getAuth } from 'firebase/auth'; // Import Firebase Authentication


export default {
  data() {
    return {
      products: [],
      itemList: [],
    };
  },
  async created() {
    try {
      const categoriesSnapshot = await getDocs(collection(firestore, 'Categories'));
      this.itemList = categoriesSnapshot.docs.map(doc => ({
        id: doc.id,
        ProductType: doc.data().ProductType,
      }));

      const productsSnapshot = await getDocs(collection(firestore, 'Products'));
      this.products = productsSnapshot.docs.map(doc => ({
        id: doc.id,
        name: doc.data()['ProductName'],
        price: doc.data().Price,
        image: doc.data().Image,
        rating: doc.data().Rating || 0,
        soldQuantity: doc.data().Sold || 0, // Fetch sold quantity from Firestore
      }));
    } catch (error) {
      console.error("Error fetching data: ", error);
    }
  },
  methods: {
    handleItemClick(item) {
      this.$router.push({ path: '/gallery', query: { category: item.ProductType } });
    },
    async addToCart(product) {
      try {
        const user = getAuth().currentUser; // Get the currently authenticated user
        const cartRef = collection(firestore, 'Cart');

        // Find if the product already exists in the cart
        const cartQuery = user
          ? query(cartRef, where("userID", "==", user.uid), where("ProductID", "==", product.id))
          : query(cartRef, where("anonymousID", "==", localStorage.getItem('anonymousID')), where("ProductID", "==", product.id));

        const cartSnapshot = await getDocs(cartQuery);

        if (!cartSnapshot.empty) {
          // If the product already exists, update its quantity
          const cartItemDoc = cartSnapshot.docs[0];
          const currentQuantity = cartItemDoc.data().Quantity;
          const updatedQuantity = currentQuantity + 1;

          await updateDoc(cartItemDoc.ref, {
            Quantity: updatedQuantity,
          });

          // console.log(`${product.name} quantity updated to ${updatedQuantity}!`);
        } else {
          // If the product doesn't exist, add it as a new item
          const cartItem = {
            ProductID: product.id,
            Quantity: 1,
          };

          if (user) {
            // If the user is signed in, link the cart item to their userID
            await addDoc(cartRef, {
              ...cartItem,
              userID: user.uid, // Add userID to the cart item
            });
            console.log(`${product.name} added to cart for user ${user.uid}!`);
          } else {
            // If no user is signed in, generate an anonymous ID
            const anonymousID = localStorage.getItem('anonymousID') || 'anon_' + Math.random().toString(36).substr(2, 9);

            localStorage.setItem('anonymousID', anonymousID);
            await addDoc(cartRef, {
              ...cartItem,
              anonymousID: anonymousID, // Add anonymousID field for non-signed-in users
            });
            console.log(`${product.name} added to cart for anonymous user ${anonymousID}!`);
          }
        }
      } catch (error) {
        console.error("Error adding to cart: ", error);
      }
    },
    goToProduct(productId) {
      this.$router.push(`/product/${productId}`);
    },
    goToGallery() {
      this.$router.push(`/gallery`);
    },
    buyNow(product) {
      // Navigate to cart page after adding the product to the cart
      this.addToCart(product).then(() => {
        this.$router.push('/cart');
      });
    },
  },
};
</script>

<style scoped>
.header-title {
  font-size: 32px;
  text-align: center;
  position: relative;
  margin-bottom: 20px;
}

.header-title::after {
  content: '';
  display: block;
  width: 60px;
  height: 4px;
  background-color: #FFA900;
  margin: 0 auto;
  margin-top: 10px;
}

.oval-button {
  border-radius: 25px;
  background-color: white;
  color: black;
  border: 1px solid black;
  transition: background-color 0.3s, color 0.3s;
}

.oval-button:hover {
  background-color: #FFA900;
  color: white;
}

.hover-card {
  transition: transform 0.3s;
  border-radius: 8px;
  overflow: hidden;
}

.hover-card:hover {
  transform: scale(1.05);
}

.cart-icon {
  margin-left: auto;
  background-color: transparent;
}

.sold-info {
  font-size: 14px;
  /* Adjust font size as needed */
  color: gray;
  /* Optional: change color to differentiate */
  margin-top: -27px;
  /* Space between name and sold quantity */
}

.see-more {
  margin-left: 7px;
  font-size: 14px;
  cursor: pointer;
}

.buy-now-btn {
  color: white;
  background-color: #FFA900;
  font-weight: bold;
  border-radius: 8px;
}

.see-more-btn {
  color: white;
  border-radius: 25px;
}
</style>
