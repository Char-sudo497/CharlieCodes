<template>
  <v-container>
    <!-- Breadcrumbs with Router Links -->
    <v-breadcrumbs class="mt-4">
      <v-breadcrumbs-item>
        <v-btn text class="breadcrumb-btn" to="/">
          Home
        </v-btn>
      </v-breadcrumbs-item>
      <v-breadcrumbs-item>
        <v-btn text class="breadcrumb-btn" to="/gallery">
          Products
        </v-btn>
      </v-breadcrumbs-item>
      <v-breadcrumbs-item>
        <v-btn text class="breadcrumb-btn" to="/aboutus">
          About Us
        </v-btn>
      </v-breadcrumbs-item>
      <v-breadcrumbs-item>
        <v-btn text class="breadcrumb-btn" to="/contactus">
          Contact Us
        </v-btn>
      </v-breadcrumbs-item>
    </v-breadcrumbs>

    <!-- Product Section -->
    <v-row v-if="product" class="mt-2 product-section">
      <!-- Product Image -->
      <v-col cols="12" md="6">
        <v-img :src="product.Image" height="400px" contain></v-img>

        <!-- Small Images -->
        <v-row class="mt-2" justify="space-between">
          <v-col cols="4" v-for="(img, index) in smallImages" :key="index">
            <v-img :src="img" height="100px" contain class="hoverable-image"></v-img>
          </v-col>
        </v-row>
      </v-col>

      <!-- Product Details -->
      <v-col cols="12" md="6">
        <v-card flat class="pa-0 d-flex flex-column justify-space-between">
          <div>
            <!-- Product Name and Rating -->
            <v-row class="align-center mb-2">
              <v-col class="d-flex align-center" style="margin-bottom: -55px;">
                <v-card-title class="text-h5 font-weight-bold">{{ product.ProductName }}</v-card-title>
                <v-icon style="font-size: 18px;" color="yellow darken-3" class="ml-2">mdi-star</v-icon>
                <span>{{ product.Rating }} ({{ product.NumberOfRatings }} ratings)</span>
              </v-col>
            </v-row>

            <!-- Sold Details -->
            <v-row>
              <v-col>
                <v-card-subtitle class="text-subtitle-1">Sold: {{ soldQuantity }} units</v-card-subtitle>
              </v-col>
            </v-row>

            <!-- Product Price and Description -->
            <v-card-subtitle class="text-h6 text--primary">₱{{ product.Price }}</v-card-subtitle>
            <v-card-text class="mt-1">{{ product.ProductDescription }}</v-card-text>

            <!-- Aluminum Size Options -->
            <v-card-title class="mt-4">Available Aluminum Sizes</v-card-title>
            <v-simple-table>
              <thead>
                <tr>
                  <th class="text-left">Size</th>
                  <th class="text-left">Thickness (mm)</th>
                  <th class="text-left">Weight (kg)</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="(size, index) in aluminumSizes" :key="index">
                  <td>{{ size.size }}</td>
                  <td>{{ size.thickness }}</td>
                  <td>{{ size.weight }}</td>
                </tr>
              </tbody>
            </v-simple-table>
          </div>

          <!-- Quantity and Add to Cart -->
          <div class="mt-4">
            <v-row class="align-center justify-start">
              <!-- Quantity Input -->
              <v-col cols="3" class="d-flex align-center">
                <v-text-field v-model.number="quantity" type="number" min="1" :max="stockQuantity"
                  class="mx-2 quantity-input" @input="validateQuantity" hide-details outlined dense
                  label="Quantity"></v-text-field>
              </v-col>
            </v-row>

            <v-row class="mt-2">
              <!-- Add to Cart Button -->
              <v-col cols="12" class="d-flex">
                <v-btn @click="addToCart(product, quantity)" block large color="black" class="add-to-cart-btn">
                  Add to Cart
                </v-btn>
              </v-col>
              <!-- Back to Products Button -->
              <v-col cols="12" class="d-flex">
                <v-btn @click="goToProducts" outlined block large class="back-btn">
                  Back to Products
                </v-btn>
              </v-col>
            </v-row>
          </div>
        </v-card>
      </v-col>
    </v-row>

    <!-- Error Handling -->
    <v-row v-else>
      <v-col cols="12">
        <v-alert type="error" border="left" colored-border>
          Product not found.
        </v-alert>
      </v-col>
    </v-row>

    <!-- Customer Reviews Section -->
    <v-container class="mt-8">
      <v-row>
        <v-col cols="12">
          <h3 class="text-h6 font-weight-bold">Customer Reviews</h3>
          <div class="review-summary">
            <div class="rating">
              <v-icon color="yellow darken-3">mdi-star</v-icon>
              <span class="font-weight-bold">4.5 out of 5</span>
            </div>
            <div class="progress-bars">
              <v-row v-for="(rating, index) in ratingsData" :key="index" class="align-center">
                <v-col cols="1">
                  <span class="star-label">{{ rating.stars }} Stars</span>
                </v-col>
                <v-col cols="8">
                  <v-progress-linear :value="rating.percentage" height="8" color="yellow darken-3"
                    class="my-1 progress-bar-custom"></v-progress-linear>
                </v-col>
              </v-row>
              <!-- Rate Products Button -->
              <v-row class="mt-2 justify-end">
                <v-col cols="auto">
                  <v-btn @click="goToRating" class="rate-products-btn" style="background-color: #ffa900;">
                    Rate Products
                  </v-btn>
                </v-col>
              </v-row>
            </div>
          </div>
        </v-col>
      </v-row>

      <v-row v-for="(review, index) in reviews" :key="index" class="review-card">
        <v-col cols="12" class="d-flex">
          <v-avatar size="40" class="mr-2">
            <v-img :src="review.avatar" />
          </v-avatar>
          <div>
            <div class="review-header d-flex align-center">
              <span class="font-weight-bold">{{ review.name }}</span>
              <v-icon color="yellow darken-3" class="ml-2">mdi-star</v-icon>
              <span class="ml-1">{{ review.rating }}</span>
            </div>
            <p class="review-content">{{ review.comment }}</p>
          </div>
        </v-col>
      </v-row>
    </v-container>
  </v-container>
</template>

<script>
import { firestore } from '~/plugins/firebase';
import { doc, getDoc, collection, addDoc } from 'firebase/firestore';

export default {
  data() {
    return {
      product: null,
      quantity: 1,
      stockQuantity: 0,
      soldQuantity: 0,
      smallImages: [
        require('@/assets/carousel 3.jpg'),
        require('@/assets/carousel 3.jpg'),
        require('@/assets/carousel 3.jpg'),
      ],
      aluminumSizes: [
        { size: "4x8", thickness: 3, weight: 25 },
        { size: "4x10", thickness: 4, weight: 30 },
        { size: "5x10", thickness: 5, weight: 35 },
      ],
      reviews: [
        {
          name: "ERIN",
          avatar: require('@/assets/avatar 1.jpg'),
          rating: 5,
          comment: "Accurate Size, The Condition is Excellent"
        },
        {
          name: "Robert",
          avatar: require('@/assets/avatar 1.jpg'),
          rating: 5,
          comment: "Great fit for my Door Size, The Condition is Excellent."
        }
      ],
      ratingsData: [
        { stars: 5, percentage: 10 },
        { stars: 4, percentage: 10 },
        { stars: 3, percentage: 10 },
        { stars: 2, percentage: 10 },
        { stars: 1, percentage: 10 }
      ]
    };
  },
  async created() {
    const productId = this.$route.params.id; // Get the product ID from the route parameters
    try {
      const productDoc = doc(firestore, 'Products', productId); // Reference to the specific product document
      const productSnapshot = await getDoc(productDoc); // Get the document snapshot

      if (productSnapshot.exists()) {
        // Store the product data and ensure you include the ID
        this.product = {
          id: productSnapshot.id, // Add the document ID
          ...productSnapshot.data() // Spread the other product fields
        };
        this.stockQuantity = this.product.Stock; // Assuming Stock is a field in your document
        this.soldQuantity = this.product.Sold; // Assuming Sold is a field in your document
      } else {
        console.log('Product not found.'); // Handle case where product does not exist
      }
    } catch (error) {
      console.error('Error fetching product:', error); // Log any errors during fetching
    }
  },
  methods: {
    validateQuantity() {
      if (this.quantity > this.stockQuantity) {
        this.quantity = this.stockQuantity; // Reset to max stock quantity if exceeded
      }
    },
    async addToCart(product, quantity) {
      try {
        const cartRef = collection(firestore, 'Cart'); // Reference to the Cart collection
        await addDoc(cartRef, {
          ProductID: product.id, // Add Product ID here
          ProductName: product.ProductName, // Ensure you are accessing correct fields
          Price: product.Price,
          Quantity: quantity,
          TotalPrice: product.Price * quantity
        });
        console.log('Product added to cart:', product.ProductName); // Log success message
      } catch (error) {
        console.error('Error adding product to cart:', error); // Log any errors during adding
      }
    },
    goToProducts() {
      this.$router.push('/gallery'); // Navigate back to products page
    },
    goToRating() {
      this.$router.push('/rating'); // Navigate to the rating page
    }
  }
};
</script>

<style scoped>
.product-section {
  padding: 20px;
}

.hoverable-image {
  cursor: pointer;
  transition: transform 0.2s;
}
.hoverable-image:hover {
  transform: scale(1.05);
}

.add-to-cart-btn {
  background-color: #ff8f00; /* Custom color for Add to Cart button */
  color: white;
  margin-bottom: -10px;
}

.back-btn {
  color: black;
  border: 2px solid black;
}

.review-card {
  padding: 10px;
  border-bottom: 1px solid #e0e0e0;
}

.rate-products-btn {
  background-color: #ff8f00; /* Custom color for Rate Products button */
  color: white;
}
</style>
