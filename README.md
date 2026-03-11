const mongoose = require('mongoose');

const productSchema = new mongoose.Schema({
    name: { type: String, required: true },
    price: { type: Number, required: true },
    description: { type: String, required: true },
    image: { type: String, required: true }, // Admin photo upload path
    category: { type: String, required: true },
    countInStock: { type: Number, required: true, default: 0 },
    reviews: [
        {
            user: { type: mongoose.Schema.Types.ObjectId, ref: 'User' },
            name: { type: String, required: true },
            rating: { type: Number, required: true },
            comment: { type: String, required: true },
        }
    ],
    numReviews: { type: Number, default: 0 },
    isReturnable: { type: Boolean, default: true }
}, { timestamps: true });

module.exports = mongoose.model('Product', productSchema);
const express = require('express');
const multer = require('multer');
const Product = require('../models/Product');
const router = express.Router();

// Storage logic for Photos
const storage = multer.diskStorage({
    destination(req, file, cb) {
        cb(null, 'uploads/'); // Photos folder
    },
    filename(req, file, cb) {
        cb(null, `${Date.now()}-${file.originalname}`);
    }
});

const upload = multer({ storage });

// Admin Route: Add New Product
router.post('/add', upload.single('image'), async (req, res) => {
    const { name, price, description, category, countInStock } = req.body;
    
    const product = new Product({
        name,
        price,
        description,
        category,
        countInStock,
        image: `/${req.file.path}` // Storing photo path
    });

    const createdProduct = await product.save();
    res.status(201).json(createdProduct);
});

module.exports = router;
// Return request logic
router.put('/:id/return', async (req, res) => {
    const order = await Order.findById(req.params.id);
    if (order) {
        order.isReturnRequested = true;
        order.returnStatus = 'Pending';
        const updatedOrder = await order.save();
        res.json(updatedOrder);
    } else {
        res.status(404).send('Order Not Found');
    }
});
const express = require('express');
const User = require('../models/User'); // Assume User model is created
const jwt = require('jsonwebtoken');
const bcrypt = require('bcryptjs');
const router = express.Router();

// 1. SIGNUP ROUTE
router.post('/signup', async (req, res) => {
    const { name, email, password } = req.body;
    const userExists = await User.findOne({ email });

    if (userExists) return res.status(400).json({ message: 'User already exists' });

    const hashedPassword = await bcrypt.hash(password, 10);
    const user = await User.create({ name, email, password: hashedPassword });

    res.status(201).json({ _id: user._id, name: user.name, email: user.email, token: generateToken(user._id) });
});

// 2. LOGIN ROUTE
router.post('/login', async (req, res) => {
    const { email, password } = req.body;
    const user = await User.findOne({ email });

    if (user && (await bcrypt.compare(password, user.password))) {
        res.json({ _id: user._id, name: user.name, email: user.email, token: generateToken(user._id) });
    } else {
        res.status(401).json({ message: 'Invalid email or password' });
    }
});

const generateToken = (id) => {
    return jwt.sign({ id }, 'Tujha_Secret_Key', { expiresIn: '30d' });
};

module.exports = router;
import React, { useState, useEffect } from 'react';

const ProfileScreen = () => {
    const [orders, setOrders] = useState([]);
    const [user, setUser] = useState({ name: 'Rahul', email: 'rahul@example.com' });

    return (
        <div className="profile-container">
            <h2>My Profile</h2>
            <div className="user-details">
                <p><strong>Name:</strong> {user.name}</p>
                <p><strong>Email:</strong> {user.email}</p>
                <button>Edit Profile</button>
            </div>

            <hr />

            <h3>My Orders</h3>
            <table border="1" width="100%">
                <thead>
                    <tr>
                        <th>ID</th>
                        <th>DATE</th>
                        <th>TOTAL</th>
                        <th>PAID</th>
                        <th>DELIVERED</th>
                        <th>ACTION</th>
                    </tr>
                </thead>
                <tbody>
                    {orders.map(order => (
                        <tr key={order._id}>
                            <td>{order._id}</td>
                            <td>{order.createdAt.substring(0, 10)}</td>
                            <td>${order.totalPrice}</td>
                            <td>{order.isPaid ? 'Yes' : 'No'}</td>
                            <td>{order.isDelivered ? 'Yes' : 'No'}</td>
                            <td>
                                <button onClick={() => handleReturn(order._id)}>Return Item</button>
                            </td>
                        </tr>
                    ))}
                </tbody>
            </table>
        </div>
    );
};

export default ProfileScreen;
const loadRazorpay = async (amount) => {
    const options = {
        key: "YOUR_RAZORPAY_KEY", 
        amount: amount * 100, // Amount in paisa
        currency: "INR",
        name: "Mazi E-com Website",
        description: "Transaction for Order",
        handler: function (response) {
            alert("Payment Successful! ID: " + response.razorpay_payment_id);
            // Ithe backend la order status 'Paid' update karaycha
        },
        prefill: { email: "customer@example.com", contact: "9876543210" }
    };
    const rzp1 = new window.Razorpay(options);
    rzp1.open();
};
import React, { useState, useEffect } from 'react';

const ProductScreen = ({ match }) => {
    const [product, setProduct] = useState({});
    const [rating, setRating] = useState(0);
    const [comment, setComment] = useState('');

    // Review Submit Logic
    const submitReviewHandler = (e) => {
        e.preventDefault();
        const reviewData = { rating, comment };
        console.log("Review Sent:", reviewData);
        alert("Review Submitted Successfully!");
    };

    return (
        <div className="product-container">
            {/* 1. Product Image & Details */}
            <div className="product-info" style={{ display: 'flex', gap: '20px' }}>
                <img src={product.image} alt={product.name} style={{ width: '400px' }} />
                
                <div>
                    <h1>{product.name}</h1>
                    <p>Price: ₹{product.price}</p>
                    <p>Description: {product.description}</p>
                    <p>Status: {product.countInStock > 0 ? 'In Stock' : 'Out of Stock'}</p>
                    
                    <button disabled={product.countInStock === 0} className="btn-cart">
                        Add to Cart
                    </button>
                </div>
            </div>

            <hr />

            {/* 2. Customer Reviews Section */}
            <div className="reviews-section">
                <h3>Customer Reviews</h3>
                {product.reviews && product.reviews.length === 0 && <p>No Reviews Yet</p>}
                
                {product.reviews && product.reviews.map((rev) => (
                    <div key={rev._id} style={{ borderBottom: '1px solid #ccc' }}>
                        <strong>{rev.name}</strong>
                        <p>Rating: {rev.rating} Stars</p>
                        <p>{rev.comment}</p>
                    </div>
                ))}

                {/* 3. Write a Review Form */}
                <div className="write-review">
                    <h4>Write a Customer Review</h4>
                    <form onSubmit={submitReviewHandler}>
                        <label>Rating:</label>
                        <select value={rating} onChange={(e) => setRating(e.target.value)}>
                            <option value="">Select...</option>
                            <option value="1">1 - Poor</option>
                            <option value="2">2 - Fair</option>
                            <option value="3">3 - Good</option>
                            <option value="4">4 - Very Good</option>
                            <option value="5">5 - Excellent</option>
                        </select>
                        <br />
                        <label>Comment:</label>
                        <textarea 
                            value={comment} 
                            onChange={(e) => setComment(e.target.value)}
                            placeholder="Write your review here..."
                        ></textarea>
                        <br />
                        <button type="submit">Submit Review</button>
                    </form>
                </div>
            </div>
        </div>
    );
};

export default ProductScreen;
router.post('/:id/reviews', async (req, res) => {
    const { rating, comment } = req.body;
    const product = await Product.findById(req.params.id);

    if (product) {
        const review = {
            name: req.user.name, // Logged in user cha name
            rating: Number(rating),
            comment,
            user: req.user._id,
        };

        product.reviews.push(review);
        product.numReviews = product.reviews.length;
        
        // Calculate average rating
        product.rating = product.reviews.reduce((acc, item) => item.rating + acc, 0) / product.reviews.length;

        await product.save();
        res.status(201).json({ message: 'Review added' });
    } else {
        res.status(404).send('Product Not Found');
    }
});
import React, { useState, useEffect } from 'react';

const AdminDashboard = () => {
    const [products, setProducts] = useState([]);

    // Logic: Database madhun products chi list aanane
    useEffect(() => {
        // fetchProducts() function ithe call karaycha
    }, []);

    const deleteHandler = (id) => {
        if (window.confirm('Are you sure you want to delete this product?')) {
            console.log('Product Deleted with ID:', id);
            // Ithe backend chi delete API call karaychi
        }
    };

    return (
        <div className="admin-container" style={{ padding: '20px' }}>
            <div style={{ display: 'flex', justifyContent: 'space-between' }}>
                <h1>Admin Dashboard</h1>
                <button style={{ backgroundColor: 'green', color: 'white', padding: '10px' }}>
                    + Create New Product
                </button>
            </div>

            <hr />

            <h3>Manage Products</h3>
            <table border="1" width="100%" style={{ textAlign: 'left', borderCollapse: 'collapse' }}>
                <thead>
                    <tr style={{ backgroundColor: '#f4f4f4' }}>
                        <th>ID</th>
                        <th>NAME</th>
                        <th>PRICE</th>
                        <th>CATEGORY</th>
                        <th>ACTIONS</th>
                    </tr>
                </thead>
                <tbody>
                    {products.map((product) => (
                        <tr key={product._id}>
                            <td>{product._id}</td>
                            <td>{product.name}</td>
                            <td>₹{product.price}</td>
                            <td>{product.category}</td>
                            <td>
                                <button onClick={() => console.log('Edit', product._id)}>Edit</button>
                                <button 
                                    onClick={() => deleteHandler(product._id)} 
                                    style={{ marginLeft: '10px', color: 'red' }}
                                >
                                    Delete
                                </button>
                            </td>
                        </tr>
                    ))}
                </tbody>
            </table>

            <hr />

            {/* Order Management Section */}
            <h3>Recent Customer Orders</h3>
            <p>(Ithe tula sarv customers ni kelelya orders chi list disel jithe tu status 'Delivered' karu shakto)</p>
        </div>
    );
};

export default AdminDashboard;
// Admin Route: Delete Product
router.delete('/:id', async (req, res) => {
    const product = await Product.findById(req.params.id);

    if (product) {
        await product.remove();
        res.json({ message: 'Product removed' });
    } else {
        res.status(404).send('Product Not Found');
    }
});
