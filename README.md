website/
│
├── index.html        # Homepage
├── books.html        # Books Listing Page
├── book-detail.html  # Single Book Detail Page
├── login.html        # User Login Page
├── register.html     # User Registration Page
├── css/
│   └── styles.css    # Styles for the website
├── js/
│   └── main.js       # JavaScript for interactive effects
├── backend/
│   ├── app.js        # Main Node.js server file
│   ├── models/
│   │   └── book.js   # MongoDB Schema for books
│   └── routes/
│       ├── books.js  # Book-related routes (search, listing)
│       └── auth.js   # Authentication routes (login, register)
├── package.json      # Node.js dependencies
└── config/
    └── database.js   # MongoDB connection configuration
    <!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Textbooks Platform - Homepage</title>
    <link rel="stylesheet" href="css/styles.css">
</head>
<body>
    <header>
        <h1>Discover Millions of Textbooks</h1>
        <input type="text" placeholder="Search for textbooks..." id="search-bar">
        <button id="search-button">Search</button>
    </header>

    <section id="featured-books">
        <h2>Featured Free Textbooks</h2>
        <div class="book-list">
            <div class="book-item">
                <img src="images/sample-book.jpg" alt="Sample Book">
                <p>Book Title 1 - Free</p>
            </div>
            <div class="book-item">
                <img src="images/sample-book.jpg" alt="Sample Book">
                <p>Book Title 2 - Free</p>
            </div>
        </div>
    </section>

    <section id="paid-books">
        <h2>Top Paid Textbooks</h2>
        <div class="book-list">
            <div class="book-item">
                <img src="images/sample-book.jpg" alt="Sample Book">
                <p>Book Title 3 - $10</p>
            </div>
            <div class="book-item">
                <img src="images/sample-book.jpg" alt="Sample Book">
                <p>Book Title 4 - $15</p>
            </div>
        </div>
    </section>

    <footer>
        <p>Textbooks Platform © 2024</p>
    </footer>

    <script src="js/main.js"></script>
</body>
</html>
/* General page styling */
body {
    font-family: Arial, sans-serif;
    background-color: #f0f0f0;
    margin: 0;
    padding: 0;
}

header {
    background-color: #2c3e50;
    color: white;
    text-align: center;
    padding: 20px;
}

h1 {
    margin: 0;
}

input[type="text"], button {
    margin-top: 20px;
    padding: 10px;
    font-size: 16px;
    width: 80%;
}

button {
    background-color: #2980b9;
    color: white;
    border: none;
    cursor: pointer;
}

button:hover {
    background-color: #3498db;
}

.book-list {
    display: flex;
    justify-content: space-around;
    margin: 20px 0;
}

.book-item {
    background-color: white;
    padding: 10px;
    text-align: center;
    width: 150px;
    border: 1px solid #ddd;
}

footer {
    background-color: #2c3e50;
    color: white;
    text-align: center;
    padding: 10px;
    position: fixed;
    width: 100%;
    bottom: 0;
}
// main.js

// Simple search functionality (demo)
document.getElementById('search-button').addEventListener('click', function() {
    const searchTerm = document.getElementById('search-bar').value.toLowerCase();
    const books = document.querySelectorAll('.book-item');

    books.forEach(book => {
        const bookTitle = book.querySelector('p').textContent.toLowerCase();
        if (bookTitle.includes(searchTerm)) {
            book.style.display = 'block';
        } else {
            book.style.display = 'none';
        }
    });
});
npm init -y
npm install express mongoose bcryptjs jsonwebtoken body-parser
const express = require('express');
const mongoose = require('mongoose');
const bodyParser = require('body-parser');

// Routes
const bookRoutes = require('./routes/books');
const authRoutes = require('./routes/auth');

// Initialize app
const app = express();
app.use(bodyParser.json());

// Connect to MongoDB
mongoose.connect('mongodb://localhost:27017/textbooks', { useNewUrlParser: true, useUnifiedTopology: true })
    .then(() => console.log('MongoDB Connected'))
    .catch(err => console.log(err));

// Use Routes
app.use('/api/books', bookRoutes);
app.use('/api/auth', authRoutes);

// Start server
const PORT = process.env.PORT || 5000;
app.listen(PORT, () => {
    console.log(`Server running on port ${PORT}`);
});
const mongoose = require('mongoose');

const BookSchema = new mongoose.Schema({
    title: { type: String, required: true },
    author: { type: String, required: true },
    price: { type: Number, required: true },
    free: { type: Boolean, default: false }
});

module.exports = mongoose.model('Book', BookSchema);
const express = require('express');
const router = express.Router();
const Book = require('../models/book');

// @route GET api/books
// @desc Get all books
router.get('/', async (req, res) => {
    try {
        const books = await Book.find();
        res.json(books);
    } catch (err) {
        res.status(500).send('Server Error');
    }
});

// @route POST api/books
// @desc Add a new book (for admin usage)
router.post('/', async (req, res) => {
    const { title, author, price, free } = req.body;

    try {
        const newBook = new Book({
            title,
            author,
            price,
            free
        });

        const book = await newBook.save();
        res.json(book);
    } catch (err) {
        res.status(500).send('Server Error');
    }
});

module.exports = router;
const express = require('express');
const bcrypt = require('bcryptjs');
const jwt = require('jsonwebtoken');
const router = express.Router();
const User = require('../models/user');

// @route POST api/auth/register
// @desc Register new user
router.post('/register', async (req, res) => {
    const { name, email, password } = req.body;

    try {
        let user = await User.findOne({ email });
        if (user) {
            return res.status(400).json({ msg: 'User already exists' });
        }

        user = new User({
            name,
            email,
            password
        });

        const salt = await bcrypt.genSalt(10);
        user.password = await bcrypt.hash(password, salt);

        await user.save();

        const payload = {
            user: {
                id: user.id
            }
        };

        jwt.sign(
            payload,
            'mysecrettoken', // Use env variable in production
            { expiresIn: 360000 },
            (err, token) => {
                if (err) throw err;
                res.json({ token });
            }
        );
    } catch (err) {
        res.status(500).send('Server Error');
    }
});

module.exports = router;
const mongoose = require('mongoose');

const connectDB = async () => {
    try {
        await mongoose.connect(process.env.MONGO_URI, {
            useNewUrlParser: true,
            useUnifiedTopology: true
