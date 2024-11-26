<template>
  <div>
    <div class="checkout-container">
      <!-- Left Column (Cart Items) -->
      <div class="checkout-left">
        <h2>Checkout</h2>

        <!-- Cart Items -->
        <div v-for="item in cartItems" :key="item.id" class="cart-item">
          <img :src="item.image || defaultImage" alt="Product Image" class="cart-item-image" />
          <div class="cart-item-details">
            <p><strong>{{ item.productName }}</strong></p>
            <p class="price">Price: ₱{{ item.price.toFixed(2) }}</p>
            <p>Quantity: {{ item.Quantity }}</p>
            <p class="total">Total: ₱{{ (item.price * item.Quantity).toFixed(2) }}</p>
          </div>
        </div>

        <!-- Delivery Address Section -->
        <div class="delivery-section">
          <h3>Delivery Address</h3>
          <p>{{ selectedAddress ? `${selectedAddress.street}, ${selectedAddress.city}, ${selectedAddress.province},
            ${selectedAddress.zip}` : 'No address selected' }}</p>
          <button @click="editAddress">Address Options</button>
        </div>

        <div class="cart-details">
          <p><strong style="font-size: 20px;">Arrives by:</strong> {{ estimatedDeliveryDate }}</p>
        </div>

        <!-- Payment Section -->
        <div class="payment-section">
          <h3>Payment</h3>
          <p v-if="selectedPaymentMethod">{{ selectedPaymentMethod.method }}</p>
          <p v-else>No Payment Method Selected</p>
          <button @click="showPaymentModal = true">Payment Options</button>
        </div>
      </div>

      <!-- Right Column (Order Summary) -->
      <div class="order-summary">
        <h3>Order Summary</h3>
        <div class="summary-item">
          <p>Subtotal:</p>
          <p>₱{{ subtotal.toFixed(2) }}</p>
        </div>
        <div class="summary-item">
          <p>Shipping Fee:</p>
          <p>₱{{ tax.toFixed(2) }}</p>
        </div>
        <hr />
        <div class="total-item">
          <p><strong>Total</strong></p>
          <p><strong>₱{{ total.toFixed(2) }}</strong></p>
        </div>
        <button class="checkout-button" @click="placeOrder">Place Your Order</button>
        <button class="back-to-cart-button" @click="goBackToCart">Back to Cart</button>
      </div>
    </div>

    <!-- Modal for Address Selection -->
    <div v-if="showAddressModal" class="address-modal">
      <div class="modal-content">
        <h3>Select a Delivery Address</h3>
        <div v-for="(address, index) in addresses" :key="index" class="address-item">
          <input type="radio" :id="'address' + index" v-model="selectedAddress" :value="address" />
          <label :for="'address' + index" class="address-label">
            {{ address.street }}, {{ address.city }}, {{ address.province }}, {{ address.zip }}
          </label>
          <button class="set-default-button" @click="setDefaultAddress(address.id)"
            :disabled="address.id === defaultAddress?.id">
            {{ address.id === defaultAddress?.id ? "Default" : "Set as Default" }}
          </button>
          <button v-if="address.id === defaultAddress?.id" class="remove-default-button"
            @click="unsetDefaultAddress(address.id)">
            Remove Default
          </button>
        </div>

        <!-- New Address Form -->
        <div class="new-address">
          <h4>Add a New Address</h4>
          <input type="text" v-model="newAddress.street" placeholder="Street Address" class="address-input" />
          <input type="text" v-model="newAddress.city" placeholder="City" class="address-input" />
          <input type="text" v-model="newAddress.province" placeholder="Province" class="address-input" />
          <input type="text" v-model="newAddress.zip" placeholder="Zip Code" class="address-input" />
          <button @click="saveAddress" class="save-button">Save Address</button>
        </div>

        <!-- Action Buttons -->
        <button class="use-button" @click="useAddress">Use Address</button>
        <!-- Move the delete button below the use address button -->
        <button v-if="selectedAddress" @click="deleteAddress(selectedAddress.id)" class="delete-button">Delete</button>
        <button class="close-button" @click="closeAddressModal">Close</button>
      </div>
    </div>

    <!-- Modal for Payment Options -->
    <div v-if="showPaymentModal" class="payment-modal">
      <div class="modal-content">
        <h3>Payment Methods</h3>

        <!-- Default Payment Methods -->
        <div v-for="method in paymentMethods" :key="method.id" class="payment-item">
          <input type="radio" :id="'method' + method.id" v-model="selectedPaymentMethod" :value="method"
            class="payment-checkbox" />
          <label :for="'method' + method.id">{{ method.method }}</label>
        </div>

        <!-- 'Others' Header for Additional Methods -->
        <h4>Others:</h4>
        <div v-for="method in otherPaymentMethods" :key="method.id" class="payment-item">
          <input type="radio" :id="'method' + method.id" v-model="selectedPaymentMethod" :value="method"
            class="payment-checkbox" />
          <label :for="'method' + method.id">{{ method.method }}</label>
        </div>

        <button @click="usePaymentMethod" class="use-button">Use</button>
        <button @click="closePaymentModal" class="close-button">Close</button>
      </div>
    </div>


  </div>
</template>

<script>
import { firestore } from '~/plugins/firebase';
import { collection, getDocs, addDoc, deleteDoc, doc, query, where, updateDoc } from 'firebase/firestore';
import { getAuth, onAuthStateChanged } from 'firebase/auth';

export default {
  data() {
    return {
      cartItems: [],
      deliveryAddress: ' ',
      subtotal: 0,
      tax: 0,
      total: 0,
      estimatedDeliveryDate: '',
      defaultImage: 'https://via.placeholder.com/60',
      selectedAddress: null,
      showAddressModal: false,
      newAddress: {
        province: '', city: '', street: '', zip: ''
      },
      defaultAddress: null,
      addresses: [],
      selectedPaymentMethod: null,  // Track the selected payment method
      showPaymentModal: false, // Track whether the payment modal is open
      paymentMethods: [], // Store available payment methods
      otherPaymentMethods: [], // Store "Others" payment methods (e.g., "Pick up")
      authInitialized: false, // Track user authentication state
    };
  },
  created() {
    this.loadCartItems();
    this.estimatedDeliveryDate = this.calculateEstimatedDeliveryDate();
    this.loadAddresses();
    this.loadPaymentMethods();
    const auth = getAuth();
    onAuthStateChanged(auth, (user) => {
      if (user) {
        this.authInitialized = true;
      } else {
        this.authInitialized = false;
      }
    });
  },
  methods: {
    async loadCartItems() {
      const items = JSON.parse(this.$route.query.items || '[]');
      this.cartItems = items;
      this.calculateTotals();
    },
    calculateTotals() {
      this.subtotal = this.cartItems.reduce((total, item) => total + (item.price * item.Quantity), 0);
      this.tax = this.subtotal * 0.1;
      this.total = this.subtotal + this.tax;
    },
    calculateEstimatedDeliveryDate() {
      const estimatedDate = new Date();
      estimatedDate.setDate(new Date().getDate() + 3);
      return estimatedDate.toDateString();
    },
    async loadAddresses() {
      const auth = getAuth();
      const user = auth.currentUser;
      if (user) {
        const userId = user.uid;
        const addressesRef = collection(firestore, 'Address');
        const querySnapshot = await getDocs(addressesRef);
        this.addresses = querySnapshot.docs
          .map(doc => ({ id: doc.id, ...doc.data() }))
          .filter(address => address.userID === userId);

        // Check for the default address
        this.defaultAddress = this.addresses.find(address => address.isDefault);
        if (this.defaultAddress) {
          this.selectedAddress = this.defaultAddress;
        }
      } else {
        console.error("User not authenticated.");
      }
    },
    async setDefaultAddress(addressId) {
      const auth = getAuth();
      const user = auth.currentUser;

      if (user) {
        const userId = user.uid;

        // Unset current default address if it exists
        if (this.defaultAddress) {
          const currentDefaultRef = doc(firestore, 'Address', this.defaultAddress.id);
          await updateDoc(currentDefaultRef, { isDefault: false });
        }

        // Set the selected address as default
        const newDefaultRef = doc(firestore, 'Address', addressId);
        await updateDoc(newDefaultRef, { isDefault: true });

        // Reload addresses to reflect changes
        this.loadAddresses();
      } else {
        console.error("User not authenticated.");
      }
    },
    async unsetDefaultAddress(addressId) {
      const auth = getAuth();
      const user = auth.currentUser;

      if (user && this.defaultAddress?.id === addressId) {
        try {
          const addressRef = doc(firestore, 'Address', addressId);
          await updateDoc(addressRef, { isDefault: false });
          this.defaultAddress = null; // Clear the current default address
          this.selectedAddress = null; // Reset the selected address if needed
          this.loadAddresses(); // Refresh the address list
        } catch (error) {
          console.error('Error unsetting default address:', error);
        }
      }
    },
    // Load all payment methods from Firestore, without filtering by user
    async loadPaymentMethods() {
      const paymentRef = collection(firestore, 'Payments');
      const querySnapshot = await getDocs(paymentRef);
      const allPaymentMethods = querySnapshot.docs.map(doc => doc.data());

      // Separate "Pick up" or other custom methods
      this.paymentMethods = allPaymentMethods.filter(method => method.method !== 'Pick up');
      this.otherPaymentMethods = allPaymentMethods.filter(method => method.method === 'Pick up');
    },
    // Use the selected payment method globally
    usePaymentMethod() {
      if (this.selectedPaymentMethod) {
        this.closePaymentModal(); // Close the modal after selection
      } else {
        alert('Please select a payment method first.');
      }
    },

    closePaymentModal() {
      this.showPaymentModal = false;
    },
    editAddress() {
      this.showAddressModal = true;
    },
    async saveAddress() {
      const auth = getAuth();
      const userId = auth.currentUser.uid;

      const newAddressData = {
        userID: userId,
        province: this.newAddress.province.trim(),
        city: this.newAddress.city.trim(),
        street: this.newAddress.street.trim(),
        zip: this.newAddress.zip.trim(),
        isDefault: false, // New addresses are not default by default
      };

      const docRef = await addDoc(collection(firestore, 'Address'), newAddressData);
      this.selectedAddress = { id: docRef.id, ...newAddressData };
      this.loadAddresses();
      this.closeAddressModal();
    },
    async deleteAddress(addressId) {
      await deleteDoc(doc(firestore, 'Address', addressId));
      this.loadAddresses(); // Refresh the list after deletion
    },
    closeAddressModal() {
      this.showAddressModal = false;
      this.newAddress = { province: '', city: '', street: '', zip: '' }; // Reset form fields
    },
    useAddress() {
      this.selectedAddress = this.addresses.find(address => address.id === this.selectedAddress.id);
      this.closeAddressModal();
    },
    async placeOrder() {
      if (!this.authInitialized) {
        console.error("User authentication is not initialized.");
        return;
      }

      if (!this.selectedAddress) {
        alert('Please select a delivery address before placing your order.');
        return; // Stop execution if no address is selected
      }

      if (!this.selectedPaymentMethod) {
        alert('Please select a payment method before placing your order.');
        return; // Stop execution if no payment method is selected
      }

      try {
        const auth = getAuth();
        const user = auth.currentUser;
        if (user) {
          const userId = user.uid;
          const orderData = {
            userId: userId,
            cartItems: this.cartItems,
            deliveryAddress: `${this.selectedAddress.province}, ${this.selectedAddress.city}, ${this.selectedAddress.street}, ${this.selectedAddress.zip}`,
            subtotal: this.subtotal,
            tax: this.tax,
            total: this.total,
            paymentMethod: this.selectedPaymentMethod.method, // Include the payment method in the order data
            estimatedDeliveryDate: this.estimatedDeliveryDate,
            status: 'Pending',
            createdAt: new Date(),
          };

          // Navigate to the Order Confirmation page, passing orderData in query params
          this.$router.push({ name: 'orderConfirmation', query: { orderData: JSON.stringify(orderData) } });
        } else {
          console.error("User not authenticated.");
        }
      } catch (error) {
        console.error("Error placing order:", error);
      }
    },
    goBackToCart() {
      this.$router.push({ name: 'cart' });
    },
  },
};
</script>

<style scoped>
.checkout-container {
  display: flex;
  flex-wrap: wrap;
  /* Allow wrapping for small screens */
  justify-content: space-between;
  gap: 20px;
  padding: 20px;
  font-family: Arial, sans-serif;
  background-color: #f9f9f9;
  border-radius: 8px;
}

.checkout-left,
.order-summary {
  flex: 1 1 100%;
  /* Full width by default */
  background-color: #fff;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.1);
}

.order-summary {
  margin-top: 20px;
  /* Add spacing between sections on small screens */
}

.cart-item {
  display: flex;
  flex-wrap: wrap;
  /* Wrap content for smaller screens */
  margin-bottom: 20px;
  align-items: center;
}

.cart-item-image {
  width: 80px;
  /* Smaller size for small screens */
  height: 80px;
  object-fit: cover;
  border-radius: 8px;
  margin-right: 15px;
}

.cart-item-details {
  flex: 1;
  /* Allow flexible width */
}

.cart-details,
.delivery-section,
.payment-section {
  margin-bottom: 20px;
}

h2,
h3 {
  font-size: 20px;
  /* Scaled font size */
  margin-bottom: 10px;
  color: #333;
}

button {
  padding: 12px;
  border-radius: 8px;
  border: none;
  cursor: pointer;
  width: 100%;
}

button:hover {
  background-color: #f4f4f4;
}

.checkout-button {
  background-color: #ff6f00;
  color: white;
  font-weight: bold;
  margin-bottom: 10px;
}

.checkout-button:hover {
  background-color: #ffa900;
}

button.back-to-cart-button {
  background-color: #f4f4f4;
  color: #ff6f00;
  font-weight: bold;
}

/* Modal Styling */
.address-modal,
.payment-modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
}

.modal-content {
  background-color: #fff;
  padding: 20px;
  width: 90%;
  /* Scale modal for small screens */
  max-width: 400px;
  border-radius: 8px;
  box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.1);
  overflow-y: auto;
}

/* Media Queries for Responsive Design */
@media (min-width: 768px) {
  .checkout-container {
    flex-wrap: nowrap;
    /* Prevent wrapping on larger screens */
  }

  .checkout-left {
    flex: 0 0 65%;
    /* Larger width for left column */
  }

  .order-summary {
    flex: 0 0 30%;
    /* Larger width for summary */
    margin-top: 0;
    /* Remove extra margin */
  }

  .cart-item-image {
    width: 120px;
    /* Larger image size */
    height: 120px;
  }

  h2,
  h3 {
    font-size: 24px;
    /* Increase font size for larger screens */
  }
}

@media (min-width: 1024px) {
  .modal-content {
    width: 400px;
    /* Fixed width for large screens */
  }
}

.remove-default-button {
  margin-left: 10px;
  background-color: #ff0000;
  color: white;
  border: none;
  padding: 5px 10px;
  border-radius: 4px;
  cursor: pointer;
}

.remove-default-button:hover {
  background-color: #cc0000;
}

.set-default-button {
  margin-left: 10px;
  background-color: #007bff;
  color: white;
  border: none;
  padding: 5px 10px;
  border-radius: 4px;
  cursor: pointer;
}

.set-default-button:disabled {
  background-color: #cccccc;
  cursor: not-allowed;
}

.checkout-container {
  display: flex;
  justify-content: space-between;
  gap: 20px;
  padding: 20px;
  font-family: Arial, sans-serif;
  background-color: #f9f9f9;
  border-radius: 8px;
}

.checkout-left {
  width: 65%;
  background-color: #fff;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.1);
}

.order-summary {
  width: 30%;
  background-color: #fff;
  padding: 20px;
  border-radius: 8px;
  border: 1px solid #e5e5e5;
  box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.1);
  position: sticky;
  top: 20px;
}

h2,
h3 {
  margin-bottom: 12px;
  font-size: 24px;
  color: #333;
}

.payment-options button {
  padding: 12px;
  border-radius: 8px;
  border: 1px solid #ccc;
  cursor: pointer;
  width: 100%;
  background-color: white;
}

.payment-options button.active {
  background-color: #ff6f00;
  /* Highlight color */
  color: white;
}

.payment-options button:hover:not(.active) {
  background-color: #f4f4f4;
  /* Light hover effect */
}

.payment-options button.active:hover {
  background-color: #ff9000;
  /* Hover effect for selected button */
}

.cart-item {
  display: flex;
  margin-bottom: 20px;
  align-items: center;
}

.cart-item-image {
  width: 120px;
  height: 120px;
  object-fit: cover;
  border-radius: 8px;
  margin-right: 15px;
}

.cart-item-details {
  display: flex;
  flex-direction: column;
}

.cart-item-details p {
  margin: 5px 0;
}

.cart-details,
.delivery-section,
.payment-section {
  margin-bottom: 30px;
}

.price {
  font-weight: bold;
}

.total {
  font-weight: bold;
  color: #ff6f00;
}

button {
  padding: 12px;
  border-radius: 8px;
  border: none;
  cursor: pointer;
  width: 100%;
}

button:hover {
  background-color: #f4f4f4;
}

.checkout-button {
  margin-bottom: 10px;
  background-color: #ff6f00;
  color: white;
  font-weight: bold;
}

.checkout-button:hover {
  background-color: #ffa900;
}

button.back-to-cart-button {
  background-color: #f4f4f4;
  color: #ff6f00;
  font-weight: bold;
}

.address-modal,
.payment-modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
}

.modal-content {
  background-color: #fff;
  padding: 20px;
  width: 300px;
  border-radius: 8px;
  box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.1);
}

.new-address input,
.address-item button {
  margin-bottom: 10px;
}

.new-address input {
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.close-button {
  background-color: #f5f5f5;
  padding: 8px;
  margin-top: 10px;
  border-radius: 4px;
  cursor: pointer;
}

.close-button:hover {
  background-color: #eaeaea;
}

.use-button {
  background-color: #ff6f00;
  color: white;
  padding: 8px;
  margin-top: 10px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.save-button {
  padding: 8px;
  border: none;
  border-radius: 4px;
  background-color: #007bff;
  color: white;
  cursor: pointer;
}

/* Modal Styling */
.address-modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 999;
}

.modal-content {
  background-color: #fff;
  padding: 30px;
  width: 350px;
  max-width: 90%;
  border-radius: 12px;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.2);
  overflow-y: auto;
}

/* Address Items */
.address-item {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 15px;
}

.address-label {
  flex: 1;
  margin-left: 10px;
  font-size: 14px;
  color: #333;
}

.delete-button {
  margin-top: 10px;
  background-color: #ff6f00;
  color: white;
  font-size: 12px;
  padding: 6px 10px;
  border-radius: 8px;
  cursor: pointer;
}

.delete-button:hover {
  background-color: #ffa900;
}

/* New Address Form */
.new-address h4 {
  font-size: 16px;
  margin-bottom: 10px;
}

.address-input {
  width: 100%;
  padding: 10px;
  margin-bottom: 10px;
  border: 1px solid #ddd;
  border-radius: 6px;
  font-size: 14px;
}

.save-button {
  background-color: #007bff;
  color: white;
  padding: 10px;
  width: 100%;
  border-radius: 6px;
  font-size: 16px;
  cursor: pointer;
}

.save-button:hover {
  background-color: #0056b3;
}

/* Action Buttons */
.use-button,
.close-button {
  background-color: #ff6f00;
  color: white;
  padding: 10px;
  width: 100%;
  border-radius: 6px;
  font-size: 16px;
  cursor: pointer;
}

.use-button:hover,
.close-button:hover {
  background-color: #ffa900;
}
</style>