<template>
    <div class="Shopping">
        <h1 class="title" style="font-size: 50px;">CompShop.io</h1><br>
        <form @submit.prevent="addNewProduct" class="form">
            <div class="form-group">
                <label for="productName">Product Name:</label>
                <input type="text" id="productName" v-model="newProduct.name" required class="input-field">
            </div>
            <div class="form-group">
                <label for="productPrice">Price:</label>
                <input type="number" id="productPrice" v-model.number="newProduct.price" min="0" step="0.01" required
                    class="input-field">
            </div>
            <button type="submit" class="btn">Add Product</button>
        </form>

        <h2 class="subtitle">Product List:</h2>
        <div class="product-list">
            <div v-for="product in products" :key="product.id" class="product">
                <div v-if="product.id !== editingProductId">
                    <p class="product-info">{{ product.name }} [Price: ₱ {{ product.price }}]</p>
                    <div class="quantity">
                        <label>Quantity:</label>
                        <input type="number" v-model="product.quantityToAdd" min="1" step="1" class="quantity-input">
                    </div>
                    <button @click="addToCart(product, product.quantityToAdd)" class="btn add-to-cart">Add to
                        Cart</button>
                </div>
                <div v-else class="edit-form">
                    <div class="form-group">
                        <label>Product Name:</label>
                        <input type="text" v-model="editedProduct.name" class="input-field">
                    </div>
                    <div class="form-group">
                        <label>Price:</label>
                        <input type="number" v-model.number="editedProduct.price" min="1" step="0.01"
                            class="input-field">
                    </div>
                    <button @click="saveEdit" class="btn">Save</button>
                    <button @click="cancelEdit" class="btn cancel">Cancel</button>
                </div>
            </div>
        </div>

        <h2 class="subtitle">Cart:</h2>
        <div class="cart">
            <div v-for="(item, index) in cart" :key="index" class="cart-item">
                <p class="cart-item-info">{{ item.name }} [₱{{ item.price }}] - Quantity: {{ item.quantity }}</p>
                <div class="quantity">
                    <label>Quantity:</label>
                    <input type="number" v-model="item.quantity" min="1" @change="updateQuantity(index, item.quantity)"
                        class="quantity-input">
                </div>
                <button @click="removeFromCart(index)" class="btn remove-btn">Remove</button>
            </div>
            <p class="total">Total: ₱{{ totalPrice }}</p>
        </div>
    </div>
</template>

<script>
export default {
    data() {
        return {
            products: [
                { id: 1, name: 'Monitor', price: 5000, quantityToAdd: 1 },
                { id: 2, name: 'Mouse', price: 150, quantityToAdd: 1 },
                { id: 3, name: 'Keyboard', price: 400, quantityToAdd: 1 },
                { id: 4, name: 'Speaker', price: 200, quantityToAdd: 1 },
                { id: 5, name: 'Headset', price: 100, quantityToAdd: 1 },
                { id: 6, name: 'Microphone', price: 250, quantityToAdd: 1 }
            ],
            newProduct: {
                name: '',
                price: null
            },
            editingProductId: null,
            editedProduct: {
                id: null,
                name: '',
                price: null
            }
        };
    },
    watch: {
        products: {
            deep: true,
            handler(newProducts) {
                this.cart.forEach(cartItem => {
                    const correspondingProduct = newProducts.find(product => product.id === cartItem.id);
                    if (correspondingProduct) {
                        cartItem.price = correspondingProduct.price;
                    }
                });
            }
        }
    },
    props: ['cart', 'router'],
    computed: {
        totalPrice() {
            return this.cart.reduce((total, item) => total + (item.price * item.quantity), 0);
        }
    },
    methods: {
        addToCart(product, quantity) {
            const isAuthenticated = localStorage.getItem('token');
            if (isAuthenticated) {
                let existingItem = this.cart.find(item => item.id === product.id);
                if (existingItem) {
                    existingItem.quantity += quantity;
                } else {
                    this.$emit('add-to-cart', { id: product.id, name: product.name, price: product.price, quantity });
                }
            } else {
                this.$emit('navigate-to-login');
            }
        },
        removeProduct(id) {
            const index = this.products.findIndex(product => product.id === id);
            if (index !== -1) {
                this.products.splice(index, 1);
            }
        },
        removeFromCart(index) {
            this.$emit('remove-from-cart', index);
        },
        updateQuantity(index, quantity) {
            if (quantity < 1) {
                this.$emit('remove-from-cart', index);
            }
        },
        addNewProduct() {
            if (this.newProduct.name && this.newProduct.price) {
                this.products.push({
                    id: this.products.length + 1,
                    name: this.newProduct.name,
                    price: parseFloat(this.newProduct.price),
                    quantityToAdd: 1
                });
                this.newProduct.name = '';
                this.newProduct.price = null;
            }
        }
    }
}
</script>

<style scoped>
.Shopping {
    font-family: 'Arial', sans-serif;
    max-width: 1000px;
    margin: 0 auto;
    padding: 20px;
    background-color: #4E89AE;
    border: 1px solid #000000;
    border-radius: 8px;
}

.title {
    text-align: center;
    font-size: 24px;
    margin-bottom: 20px;
}

.subtitle {
    font-size: 20px;
    margin-top: 30px;
    margin-bottom: 15px;
}

.form-group {
    margin-bottom: 15px;
}

.label {
    font-weight: bold;
}

.input-field {
    width: 150px;
    padding: 8px;
    border-radius: 5px;
    border: 1px solid #ccc;
}

.form-group {
    font-weight: bold;
    max-width: auto;
}

.btn {
    padding: 10px 20px;
    border: none;
    border-radius: 5px;
    cursor: pointer;
    color: #050505;
}

.btn:hover {
    opacity: 0.8;
}

.add-to-cart {
    background-color: #00A9FF;
    border-radius: 5px;
    font-weight: bold;
}

.remove-btn {
    background-color: #F49D1A;
    border-radius: 5px;
    font-weight: bold;
}

.cancel {
    background-color: #3498db;
    border-radius: 5px;
    font-weight: bold;
}

.product-list {
    display: flex;
    flex-wrap: wrap;
    justify-content: space-between;
}

.product {
    width: 45%;
    border: 1px solid #ccc;
    border-radius: 8px;
    padding: 15px;
    margin-bottom: 20px;
    background-color: #ffffffc6;
}

.edit-form {
    padding: 15px;
    border: 1px solid #ccc;
    border-radius: 8px;
    background-color: #f9f9f9;
    margin-bottom: 20px;
}

.cart {
    border: 1px solid #ccc;
    border-radius: 8px;
    padding: 15px;
    background-color: #f9f9f9;
}

.cart-item {
    border-bottom: 1px solid #ccc;
    padding-bottom: 10px;
    margin-bottom: 10px;
}

.cart-item:last-child {
    border-bottom: none;
}

.cart-item-info {
    margin-bottom: 5px;
}

.quantity {
    margin-bottom: 10px;
}

.quantity label {
    font-weight: bold;
}

.quantity-input {
    width: 70px;
    padding: 8px;
    border-radius: 5px;
    border: 1px solid #ccc;
}

.total {
    font-weight: bold;
    font-size: 20px;
    text-align: right;
}
</style>
