<template>
    <v-app>
        <v-container>
            <!-- Orders Table -->
            <v-row>
                <v-col cols="12">
                    <v-card-title class="headline text-center">
                        <v-icon class="mr-2">mdi-cube</v-icon>
                        Pending Orders
                    </v-card-title>
                    <v-data-table :headers="headers" :items="pendingOrders" item-key="id" class="elevation-1">
                        <template v-slot:item.status="{ item }">
                            <v-chip :color="getStatusColor(item.status)" outlined>
                                {{ item.status }}
                            </v-chip>
                        </template>

                        <template v-slot:item.createdAt="{ item }">
                            <span>{{ formatDate(item.createdAt) }}</span>
                        </template>

                        <template v-slot:item.cartItems="{ item }">
                            <div v-for="(product, index) in item.cartItems" :key="index">
                                <span>{{ product.productName }}</span>
                            </div>
                        </template>

                        <template v-slot:item.totalQuantity="{ item }">
                            <span>{{ calculateTotalQuantity(item.cartItems) }}</span>
                        </template>

                        <template v-slot:item.totalAmount="{ item }">
                            <span>₱{{ item.totalAmount || 'Not Available' }}</span>
                        </template>

                        <template v-slot:item.userId="{ item }">
                            <span>{{ item.userFullName }}</span> <!-- Display the user's full name -->
                        </template>

                        <template v-slot:item.actions="{ item }">
                            <v-btn @click="updateOrderStatus(item)" color="blue" style="color: white;">Update
                                Status</v-btn>
                            <v-btn @click="viewOrderDetails(item)" color="green" style="color: white;">View
                                Details</v-btn>
                        </template>
                    </v-data-table>
                </v-col>
            </v-row>

            <!-- Order Details Dialog -->
            <v-dialog v-model="detailsDialog" max-width="500px">
                <v-card>
                    <v-card-title class="headline">Order Details</v-card-title>
                    <v-card-text>
                        <div v-if="selectedOrderDetails">
                            <p><strong>User ID:</strong> {{ selectedOrderDetails.userId }}</p>
                            <p><strong>Product Name:</strong> {{ selectedOrderDetails.productName }}</p>
                            <p><strong>Quantity:</strong> {{ selectedOrderDetails.Quantity }}</p>
                            <p><strong>Subtotal:</strong> ₱{{ selectedOrderDetails.subtotal || 'Not Available' }}</p>
                            <p><strong>Tax:</strong> ₱{{ selectedOrderDetails.tax || 'Not Available' }}</p>
                            <p><strong>Total:</strong> ₱{{ selectedOrderDetails.total }}</p>
                            <p><strong>Payment Method:</strong> {{ selectedOrderDetails.paymentMethod }}</p>
                            <p><strong>Delivery Address:</strong> {{ selectedOrderDetails.deliveryAddress || 'Not Available' }}</p>
                            <p><strong>Estimated Delivery Date:</strong> {{ selectedOrderDetails.estimatedDeliveryDate
                                }}
                            </p>
                            <p><strong>Status:</strong> {{ selectedOrderDetails.status }}</p>
                        </div>
                        <v-btn @click="detailsDialog = false" color="grey">Close</v-btn>
                    </v-card-text>
                </v-card>
            </v-dialog>

            <!-- Shipped Orders Table -->
            <v-row>
                <v-col cols="12">
                    <v-card-title class="headline text-center">
                        <v-icon class="mr-2">mdi-cube</v-icon>
                        Shipped Orders
                    </v-card-title>
                    <v-data-table :headers="headers" :items="shippedOrders" item-key="id" class="elevation-1">
                        <template v-slot:item.status="{ item }">
                            <v-chip :color="getStatusColor(item.status)" outlined>
                                {{ item.status }}
                            </v-chip>
                        </template>

                        <template v-slot:item.createdAt="{ item }">
                            <span>{{ formatDate(item.createdAt) }}</span>
                        </template>

                        <template v-slot:item.cartItems="{ item }">
                            <div v-for="(product, index) in item.cartItems" :key="index">
                                <span>{{ product.productName }}</span>
                            </div>
                        </template>

                        <template v-slot:item.totalQuantity="{ item }">
                            <span>{{ calculateTotalQuantity(item.cartItems) }}</span>
                        </template>

                        <template v-slot:item.totalAmount="{ item }">
                            <span>₱{{ item.totalAmount || 'Not Available' }}</span>
                        </template>

                        <template v-slot:item.userId="{ item }">
                            <span>{{ item.userFullName }}</span> <!-- Display the user's full name -->
                        </template>

                        <template v-slot:item.actions="{ item }">
                            <v-btn @click="updateOrderStatus(item)" color="blue" style="color: white;">Update
                                Status</v-btn>
                            <v-btn @click="viewOrderDetails(item)" color="green" style="color: white;">View
                                Details</v-btn>
                        </template>
                    </v-data-table>
                </v-col>
            </v-row>

            <!-- Delivered Orders Table -->
            <v-row>
                <v-col cols="12">
                    <v-card-title class="headline text-center">
                        <v-icon class="mr-2">mdi-cube</v-icon>
                        Delievered Orders
                    </v-card-title>
                    <v-data-table :headers="headers" :items="deliveredOrders" item-key="id" class="elevation-1">
                        <template v-slot:item.status="{ item }">
                            <v-chip :color="getStatusColor(item.status)" outlined>
                                {{ item.status }}
                            </v-chip>
                        </template>

                        <template v-slot:item.createdAt="{ item }">
                            <span>{{ formatDate(item.createdAt) }}</span>
                        </template>

                        <template v-slot:item.cartItems="{ item }">
                            <div v-for="(product, index) in item.cartItems" :key="index">
                                <span>{{ product.productName }}</span>
                            </div>
                        </template>

                        <template v-slot:item.totalQuantity="{ item }">
                            <span>{{ calculateTotalQuantity(item.cartItems) }}</span>
                        </template>

                        <template v-slot:item.totalAmount="{ item }">
                            <span>₱{{ item.totalAmount || 'Not Available' }}</span>
                        </template>

                        <template v-slot:item.userId="{ item }">
                            <span>{{ item.userFullName }}</span> <!-- Display the user's full name -->
                        </template>

                        <template v-slot:item.actions="{ item }">
                            <v-btn @click="updateOrderStatus(item)" color="blue" style="color: white;">Update
                                Status</v-btn>
                            <v-btn @click="viewOrderDetails(item)" color="green" style="color: white;">View
                                Details</v-btn>
                        </template>
                    </v-data-table>
                </v-col>
            </v-row>

            <!-- Completed Orders Table -->
            <v-row>
                <v-col cols="12">
                    <v-card-title class="headline text-center">
                        <v-icon class="mr-2">mdi-cube</v-icon>
                        Completed Orders
                    </v-card-title>
                    <v-data-table :headers="headers" :items="completedOrders" item-key="id" class="elevation-1">
                        <template v-slot:item.status="{ item }">
                            <v-chip :color="getStatusColor(item.status)" outlined>
                                {{ item.status }}
                            </v-chip>
                        </template>

                        <template v-slot:item.createdAt="{ item }">
                            <span>{{ formatDate(item.createdAt) }}</span>
                        </template>

                        <template v-slot:item.cartItems="{ item }">
                            <div v-for="(product, index) in item.cartItems" :key="index">
                                <span>{{ product.productName }}</span>
                            </div>
                        </template>

                        <template v-slot:item.totalQuantity="{ item }">
                            <span>{{ calculateTotalQuantity(item.cartItems) }}</span>
                        </template>

                        <template v-slot:item.totalAmount="{ item }">
                            <span>₱{{ item.totalAmount || 'Not Available' }}</span>
                        </template>

                        <template v-slot:item.userId="{ item }">
                            <span>{{ item.userFullName }}</span> <!-- Display the user's full name -->
                        </template>

                        <template v-slot:item.actions="{ item }">
                            <v-btn @click="updateOrderStatus(item)" color="blue" style="color: white;">Update
                                Status</v-btn>
                            <v-btn @click="viewOrderDetails(item)" color="green" style="color: white;">View
                                Details</v-btn>
                        </template>
                    </v-data-table>
                </v-col>
            </v-row>

            <!-- Cancelled Orders Table -->
            <v-row>
                <v-col cols="12">
                    <v-card-title class="headline text-center">
                        <v-icon class="mr-2">mdi-cube</v-icon>
                        Cancelled Orders
                    </v-card-title>
                    <v-data-table :headers="headers" :items="cancelledOrders" item-key="id" class="elevation-1">
                        <template v-slot:item.status="{ item }">
                            <v-chip :color="getStatusColor(item.status)" outlined>
                                {{ item.status }}
                            </v-chip>
                        </template>

                        <template v-slot:item.createdAt="{ item }">
                            <span>{{ formatDate(item.createdAt) }}</span>
                        </template>

                        <template v-slot:item.cartItems="{ item }">
                            <div v-for="(product, index) in item.cartItems" :key="index">
                                <span>{{ product.productName }}</span>
                            </div>
                        </template>

                        <template v-slot:item.totalQuantity="{ item }">
                            <span>{{ calculateTotalQuantity(item.cartItems) }}</span>
                        </template>

                        <template v-slot:item.totalAmount="{ item }">
                            <span>₱{{ item.totalAmount || 'Not Available' }}</span>
                        </template>

                        <template v-slot:item.userId="{ item }">
                            <span>{{ item.userFullName }}</span> <!-- Display the user's full name -->
                        </template>

                        <template v-slot:item.actions="{ item }">
                            <v-btn @click="updateOrderStatus(item)" color="blue" style="color: white;">Update
                                Status</v-btn>
                            <v-btn @click="viewOrderDetails(item)" color="green" style="color: white;">View
                                Details</v-btn>
                        </template>
                    </v-data-table>
                </v-col>
            </v-row>

            <!-- Order Status Update Form -->
            <v-dialog v-model="dialog" max-width="500px">
                <v-card>
                    <v-card-title class="headline">Update Order Status</v-card-title>
                    <v-card-text>
                        <v-select v-model="selectedStatus" :items="statusOptions" label="Select Status"
                            required></v-select>
                    </v-card-text>
                    <v-card-actions>
                        <v-btn @click="dialog = false" color="grey">Cancel</v-btn>
                        <v-btn @click="saveStatusUpdate" color="primary">Save</v-btn>
                    </v-card-actions>
                </v-card>
            </v-dialog>
        </v-container>
    </v-app>
</template>

<script>
import { firestore } from '~/plugins/firebase';
import { collection, getDocs, getDoc, doc, updateDoc } from 'firebase/firestore';

export default {
    data() {
        return {
            orders: [],
            shippedOrders: [],
            deliveredOrders: [],
            completedOrders: [],
            cancelledOrders: [],
            headers: [
                { text: 'Order ID', value: 'id' },
                { text: 'Products Ordered', value: 'cartItems' },
                { text: 'User', value: 'userId' },
                { text: 'Quantity', value: 'totalQuantity' },
                { text: 'Total', value: 'totalAmount' },
                { text: 'Status', value: 'status' },
                { text: 'Payment Method', value: 'paymentMethod' },
                { text: 'Estimated Delivery Date', value: 'estimatedDeliveryDate' },
                { text: 'Created At', value: 'createdAt' },
                { text: 'Actions', value: 'actions', sortable: false },
            ],
            dialog: false,
            detailsDialog: false,
            selectedStatus: '',
            selectedOrderDetails: null,
            statusOptions: ['Pending', 'Shipped', 'Delivered', 'Completed', 'Cancelled'],
            selectedOrderId: null,
        };
    },
    async created() {
        try {
            const ordersSnapshot = await getDocs(collection(firestore, 'Orders'));
            this.orders = [];

            // Fetch orders and their user details
            for (const docSnap of ordersSnapshot.docs) {
                const orderData = docSnap.data();
                const userRef = doc(firestore, 'Users', orderData.userId);
                const userDoc = await getDoc(userRef);
                const userData = userDoc.exists() ? userDoc.data() : null;
                const userFullName = userData ? `${userData.firstName} ${userData.lastName}` : 'Unknown';

                const order = {
                    id: docSnap.id,
                    ...orderData,
                    totalAmount: orderData.totalAmount || this.calculateTotalAmount(orderData.cartItems),
                    status: orderData.status,
                    userFullName, // Added userFullName to order
                };
                this.orders.push(order);
            }

            // Filter orders to only include those with specific statuses
            this.shippedOrders = this.orders.filter(order => order.status === 'Shipped');
            this.deliveredOrders = this.orders.filter(order => order.status === 'Delivered');
            this.completedOrders = this.orders.filter(order => order.status === 'Completed');
            this.cancelledOrders = this.orders.filter(order => order.status === 'Cancelled');
        } catch (error) {
            console.error('Error fetching orders: ', error);
        }
    },
    computed: {
        pendingOrders() {
            return this.orders.filter(order => order.status === 'Pending');
        },
    },
    methods: {
        formatDate(date) {
            const options = { year: 'numeric', month: 'short', day: 'numeric' };
            return new Date(date.seconds * 1000).toLocaleDateString('en-US', options);
        },
        getStatusColor(status) {
            switch (status) {
                case 'Pending': return 'orange';
                case 'Shipped': return 'blue';
                case 'Delivered': return 'green';
                case 'Completed': return 'teal';
                case 'Cancelled': return 'red';
                default: return 'grey';
            }
        },
        updateOrderStatus(order) {
            this.selectedOrderId = order.id;
            this.selectedStatus = order.status;
            this.dialog = true;
        },
        async saveStatusUpdate() {
            try {
                const orderRef = doc(firestore, 'Orders', this.selectedOrderId);
                await updateDoc(orderRef, {
                    status: this.selectedStatus,
                });

                this.dialog = false;

                const updatedOrder = this.orders.find(order => order.id === this.selectedOrderId);
                if (updatedOrder) {
                    updatedOrder.status = this.selectedStatus;
                }

                this.shippedOrders = this.orders.filter(order => order.status === 'Shipped');
                this.deliveredOrders = this.orders.filter(order => order.status === 'Delivered');
                this.completedOrders = this.orders.filter(order => order.status === 'Completed');
                this.cancelledOrders = this.orders.filter(order => order.status === 'Cancelled');
            } catch (error) {
                console.error('Error updating order status: ', error);
            }
        },
        calculateTotalQuantity(cartItems) {
            return cartItems.reduce((total, product) => {
                const quantity = product.Quantity || 0;
                return total + quantity;
            }, 0);
        },
        calculateTotalAmount(cartItems) {
            return cartItems.reduce((total, product) => {
                const price = product.price || 0;
                const quantity = product.Quantity || 0;
                return total + (price * quantity);
            }, 0);
        },
        viewOrderDetails(order) {
            this.selectedOrderDetails = {
                userId: order.userId,
                productName: order.cartItems.map(item => item.productName).join(', '),
                Quantity: this.calculateTotalQuantity(order.cartItems),
                subtotal: order.cartItems.reduce((sum, item) => sum + (item.price * item.Quantity), 0),
                tax: order.cartItems.reduce((sum, item) => sum + (item.tax || 0), 0),
                total: order.totalAmount,
                paymentMethod: order.paymentMethod,
                estimatedDeliveryDate: order.estimatedDeliveryDate,
                status: order.status,
                // Include the new deliveryAddress field
                deliveryAddress: order.deliveryAddress || 'Not Available',
            };
            this.detailsDialog = true;
        },
    },
};
</script>

<style scoped>
.v-data-table {
    margin-top: 20px;
}
</style>
