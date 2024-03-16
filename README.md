# Adsremed_assignment
Below is a basic implementation of the REST API for the hypothetical online restaurant service using Node.js with Express.js for creating the API endpoints and MongoDB for the database. This example covers creating, reading, updating, and deleting products, placing orders, and listing all orders.
// Import required modules
const express = require('express');
const mongoose = require('mongoose');
const bodyParser = require('body-parser');

// Initialize Express app
const app = express();

// Connect to MongoDB
mongoose.connect('mongodb://localhost/restaurant_db', { useNewUrlParser: true, useUnifiedTopology: true });
const db = mongoose.connection;
db.on('error', console.error.bind(console, 'MongoDB connection error:'));

// Define Product schema
const ProductSchema = new mongoose.Schema({
    name: String,
    description: String,
    price: Number
});
const Product = mongoose.model('Product', ProductSchema);

// Define Order schema
const OrderSchema = new mongoose.Schema({
    product: {
        type: mongoose.Schema.Types.ObjectId,
        ref: 'Product'
    },
    customerName: String,
    quantity: Number
});
const Order = mongoose.model('Order', OrderSchema);

// Middleware
app.use(bodyParser.json());

// Routes
// Product CRUD operations
app.post('/products', async (req, res) => {
    try {
        const { name, description, price } = req.body;
        const product = new Product({ name, description, price });
        await product.save();
        res.json(product);
    } catch (err) {
        res.status(500).json({ error: err.message });
    }
});

app.get('/products', async (req, res) => {
    try {
        const products = await Product.find();
        res.json(products);
    } catch (err) {
        res.status(500).json({ error: err.message });
    }
});

app.put('/products/:id', async (req, res) => {
    try {
        const { id } = req.params;
        const product = await Product.findByIdAndUpdate(id, req.body, { new: true });
        res.json(product);
    } catch (err) {
        res.status(500).json({ error: err.message });
    }
});

app.delete('/products/:id', async (req, res) => {
    try {
        const { id } = req.params;
        await Product.findByIdAndDelete(id);
        res.json({ message: 'Product deleted successfully' });
    } catch (err) {
        res.status(500).json({ error: err.message });
    }
});

// Place an order for a product
app.post('/orders', async (req, res) => {
    try {
        const { productId, customerName, quantity } = req.body;
        const product = await Product.findById(productId);
        if (!product) {
            return res.status(404).json({ error: 'Product not found' });
        }
        const order = new Order({ product: productId, customerName, quantity });
        await order.save();
        res.json(order);
    } catch (err) {
        res.status(500).json({ error: err.message });
    }
});

// List all orders
app.get('/orders', async (req, res) => {
    try {
        const orders = await Order.find().populate('product');
        res.json(orders);
    } catch (err) {
        res.status(500).json({ error: err.message });
    }
});

// Start server
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
    console.log(`Server is running on port ${PORT}`);
});
