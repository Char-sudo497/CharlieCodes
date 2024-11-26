<template>
  <div class="order-confirmation">
    <h2>Order Confirmation</h2>

    <div v-if="orderData" class="order-summary">
      <!-- Order Items Section -->
      <div class="order-items">
        <h3>Items</h3>
        <ul>
          <li v-for="item in orderData.cartItems" :key="item.id" class="order-item">
            <div class="item-image">
              <img :src="item.image" alt="Product Image" class="product-image" v-if="item.image" />
            </div>
            <div class="item-details">
              <p class="product-name"><strong>{{ item.productName }}</strong></p>
              <p>Price: ₱{{ item.price.toFixed(2) }}</p>
              <p>Quantity: {{ item.Quantity }}</p>
            </div>
          </li>
        </ul>
      </div>

      <!-- Order Details Section -->
      <div class="order-details">
        <h3>Order Details</h3>
        <p><strong>Delivery Address:</strong> {{ orderData.deliveryAddress }}</p>
        <p><strong>Estimated Delivery Date:</strong> {{ orderData.estimatedDeliveryDate }}</p>
        <p><strong>Payment Method:</strong> {{ orderData.paymentMethod }}</p>

        <!-- Separation Line -->
        <hr class="separator" />

        <!-- Subtotal, Tax, and Total Section -->
        <div class="pricing-details">
          <div>
            <p><strong>Subtotal:</strong> ₱{{ orderData.subtotal.toFixed(2) }}</p>
            <p><strong>Shipping Fee:</strong> ₱{{ orderData.tax.toFixed(2) }}</p>
            <p><strong>Total:</strong> ₱{{ orderData.total.toFixed(2) }}</p>
          </div>
        </div>
      </div>

      <!-- Action Buttons -->
      <div class="action-buttons">
        <button @click="confirmOrder" class="confirm-button">Confirm Order</button>
        <button @click="goBacktoCart" class="go-back-button">Go Back to Cart</button>
      </div>

      <!-- GCash Info Dialog -->
      <v-dialog v-model="gcashDialog" max-width="500px">
        <v-card>
          <v-card-title class="headline">GCash Payment</v-card-title>
          <v-card-text>
            <div class="text-center">
              <img src="@/assets/Gcash.jpg" alt="GCash Image" class="gcash-image" />
              <p class="gcash-description">Scan the QR code to pay via GCash.</p>
            </div>
          </v-card-text>
          <v-card-actions>
            <v-btn @click="openQrCodeDialog" color="green">Proceed to QR Code</v-btn>
            <v-btn @click="closeGcashDialog" color="red">Cancel</v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>

      <!-- QR Code Dialog -->
      <v-dialog v-model="qrCodeDialog" max-width="400px">
        <v-card>
          <v-card-title class="headline">Order QR Code</v-card-title>
          <v-card-text>
            <div class="text-center">
              <canvas ref="qrCanvas" width="200" height="200"></canvas>
              <p class="qr-description">
                You can use this QR code for pickup or as proof of your order.
              </p>
            </div>
          </v-card-text>
          <v-card-actions>
            <v-btn @click="goBack" color="green" style="color: white;">Done</v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>

    </div>
  </div>
</template>

<script>
import QRCode from 'qrcode';
import { firestore } from '~/plugins/firebase';
import { collection, addDoc, getDocs, query, where } from 'firebase/firestore';
import { getAuth } from 'firebase/auth';

export default {
  data() {
    return {
      orderData: null,
      qrCodeDialog: false, // Dialog visibility for QR code
      gcashDialog: false,  // Dialog visibility for GCash payment info
      orderId: null,       // Store the orderId after order creation
    };
  },
  created() {
    const orderData = this.$route.query.orderData;
    if (orderData) {
      this.orderData = JSON.parse(orderData);
      this.fetchProductImages();
    }
  },
  methods: {
    async fetchProductImages() {
      try {
        const productsRef = collection(firestore, 'Products');
        for (let item of this.orderData.cartItems) {
          const q = query(productsRef, where('productName', '==', item.productName));
          const querySnapshot = await getDocs(q);
          querySnapshot.forEach((doc) => {
            if (doc.exists()) {
              const productData = doc.data();
              item.image = productData.Image;
            }
          });
        }
      } catch (error) {
        console.error('Error fetching product images:', error);
      }
    },
    async confirmOrder() {
      try {
        if (this.orderData) {
          const auth = getAuth();
          const user = auth.currentUser;
          if (user) {
            const orderRef = collection(firestore, 'Orders');
            const orderDocRef = await addDoc(orderRef, {
              userId: this.orderData.userId,
              cartItems: this.orderData.cartItems,
              deliveryAddress: this.orderData.deliveryAddress,
              paymentMethod: this.orderData.paymentMethod,
              subtotal: this.orderData.subtotal,
              tax: this.orderData.tax,
              total: this.orderData.total,
              estimatedDeliveryDate: this.orderData.estimatedDeliveryDate,
              status: 'Pending',
              createdAt: new Date(),
            });

            this.orderId = orderDocRef.id;
            this.orderData.orderId = this.orderId;

            // Check if payment method is GCash
            if (this.orderData.paymentMethod === 'GCash') {
              this.openGcashDialog(); // Open GCash dialog
            } else {
              this.openQrCodeDialog(); // Proceed to QR code directly
            }
          } else {
            console.error('User not authenticated.');
          }
        }
      } catch (error) {
        console.error('Error confirming order:', error);
      }
    },
    openGcashDialog() {
      this.gcashDialog = true; // Show GCash dialog
    },
    closeGcashDialog() {
      this.gcashDialog = false; // Close GCash dialog
    },
    openQrCodeDialog() {
      this.gcashDialog = false; // Close GCash dialog
      this.qrCodeDialog = true; // Show QR code dialog
      this.$nextTick(() => {
        this.generateQrCode();
      });
    },
    closeQrCodeDialog() {
      this.qrCodeDialog = false;
    },
    async generateQrCode() {
      try {
        const usersRef = collection(firestore, 'Users');
        const q = query(usersRef, where('__name__', '==', this.orderData.userId));
        const querySnapshot = await getDocs(q);

        let userName = 'Unknown User';
        querySnapshot.forEach((doc) => {
          if (doc.exists()) {
            const userDetails = doc.data();
            userName = `${userDetails.firstName || ''} ${userDetails.lastName || ''}`.trim() || 'Unknown User';
          }
        });

        const orderDetails = {
          orderId: this.orderData.orderId,
          items: this.orderData.cartItems.map(item => ({
            productName: item.productName,
            quantity: item.Quantity,
            total: this.orderData.total,
          }))
        };

        const qrData = JSON.stringify(orderDetails);
        const canvas = this.$refs.qrCanvas;

        if (canvas) {
          const ctx = canvas.getContext('2d');
          ctx.clearRect(0, 0, canvas.width, canvas.height);
          await QRCode.toCanvas(canvas, qrData, {
            width: 200,
            margin: 4,
          });
        }
      } catch (error) {
        console.error('Error generating QR code:', error);
      }
    },
    goBacktoCart() {
      this.$router.push({ name: 'orderConfirmationSuccess' });
    },
    goBack() {
      this.$router.push({ name: 'orderConfirmationSuccess' });
    },
  },
};
</script>

<style scoped>
.order-confirmation {
  padding: 20px;
  background-color: #fff;
  border-radius: 8px;
}

.gcash-image {
  width: 100%;
  height: auto;
  max-width: 300px;
  margin-bottom: 10px;
}

.gcash-description {
  font-size: 1.2rem;
  color: #444;
}

/* Styling buttons and dialogs */
.separator {
  border: none;
  border-top: 2px solid #ddd;
  margin: 20px 0;
}

.order-confirmation {
  padding: 20px;
  background-color: #fff;
  border-radius: 8px;
}

.order-summary {
  max-width: 1200px;
  margin: 0 auto;
}

h2 {
  font-size: 2rem;
  color: #333;
  margin-bottom: 20px;
}

h3 {
  margin-bottom: 15px;
  font-size: 1.5rem;
  color: #444;
}

.order-items,
.order-details {
  background-color: white;
  border-radius: 8px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  padding: 20px;
  margin-bottom: 20px;
}

.order-item {
  display: flex;
  margin-bottom: 20px;
  border-bottom: 1px solid #ddd;
  padding-bottom: 15px;
}

.item-image {
  flex: 1;
  max-width: 120px;
}

.product-image {
  width: 100%;
  border-radius: 8px;
  object-fit: cover;
}

.item-details {
  flex: 2;
  padding-left: 20px;
}

.item-details p {
  margin: 5px 0;
}

.details-row {
  display: flex;
  justify-content: space-between;
  margin-bottom: 10px;
}

.pricing-details {
  margin-top: 20px;
}

.pricing-details .total p {
  font-size: 1.2rem;
  font-weight: bold;
  color: #d9534f;
}

/* Styling the separation line */
.separator {
  border: none;
  border-top: 2px solid #ddd;
  margin: 20px 0;
}

button {
  padding: 12px 20px;
  font-size: 1rem;
  border-radius: 5px;
  cursor: pointer;
  border: none;
}

.confirm-button {
  background-color: #28a745;
  color: white;
  margin-right: 15px;
}

.confirm-button:hover {
  background-color: #218838;
}

.go-back-button {
  background-color: #007bff;
  color: white;
}

.go-back-button:hover {
  background-color: #0056b3;
}

.qr-description {
  margin-top: 10px;
  font-size: 1rem;
  color: #444;
}
</style>
