/ server.js (In-memory data storage)

const express = require('express');
const path = require('path');
const multer = require('multer'); // For handling file uploads (e.g., product images)
const cors = require('cors'); // For allowing cross-origin requests from your frontend
const fs = require('fs'); // Node.js File System module

const app = express();
const PORT = process.env.PORT || 3000;

// --- In-Memory Data Stores ---
const users = []; // Stores user objects: { username, email, password }
const products = []; // Stores product objects: { id, title, category, condition, price, description, imageUrl }

// --- Middleware ---
// Configure CORS for development. In production, restrict to your actual domain.
app.use(cors({
  origin: 'http://localhost:3000', // Allow requests from your own frontend
  methods: ['GET', 'POST'],
  allowedHeaders: ['Content-Type']
}));
app.use(express.json()); // Parses incoming JSON requests
app.use(express.urlencoded({ extended: true })); // Parses URL-encoded data (for forms)

// Serve static files (your frontend HTML, CSS, JS) from the 'public' directory
app.use(express.static(path.join(__dirname, 'public')));

// Create the uploads directory if it doesn't exist
const uploadsDir = path.join(__dirname, 'public', 'uploads');
if (!fs.existsSync(uploadsDir)) {
    fs.mkdirSync(uploadsDir, { recursive: true });
}

// --- Multer for Image Uploads ---
// This stores images locally. For production, use cloud storage (AWS S3, Cloudinary).
const storage = multer.diskStorage({
  destination: (req, file, cb) => {
    cb(null, uploadsDir); // Images will be saved in public/uploads/
  },
  filename: (req, file, cb) => {
    // Generate a unique filename (timestamp + original extension)
    cb(null, Date.now() + path.extname(file.originalname));
  }
});
const upload = multer({ storage: storage });

// --- API Endpoints ---

// Route to get all products (for the "Buy" section)
app.get('/api/products', (req, res) => {
  // Return all products currently in memory
  res.json(products);
});

// Route to add a new product (for the "Sell" section)
// 'upload.single('image')' handles the file upload for a field named 'image'
app.post('/api/products', upload.single('image'), (req, res) => {
  try {
    const { title, category, condition, price, description } = req.body;
    
    // Construct the image URL. Multer saves the file, we just need the path.
    const imageUrl = req.file ? `/uploads/${req.file.filename}` : 'https://via.placeholder.com/250x250/F0F0F0/000000?text=No+Image';

    if (!title || !category || !condition || !price) {
        // If an image was uploaded but other data is missing, clean up the uploaded file
        if (req.file) {
            fs.unlink(req.file.path, (err) => {
                if (err) console.error('Error deleting incomplete upload:', err);
            });
        }
        return res.status(400).json({ message: 'Missing required product fields.' });
    }

    const newProduct = {
      id: products.length > 0 ? Math.max(...products.map(p => p.id)) + 1 : 1, // Simple auto-incrementing ID
      title,
      category,
      condition,
      price: parseFloat(price), // Convert price to number
      description,
      imageUrl,
      createdAt: new Date()
    };

    products.push(newProduct); // Add the new product to the in-memory array
    console.log('New product listed (in-memory):', newProduct);
    res.status(201).json({ message: 'Product listed successfully!', product: newProduct });
  } catch (error) {
    console.error('Failed to list product:', error);
    // If an error occurred after file upload but before saving product data, try to delete the file
    if (req.file) {
        fs.unlink(req.file.path, (err) => {
            if (err) console.error('Error deleting failed upload:', err);
        });
    }
    res.status(500).json({ message: 'Failed to list product', error: error.message });
  }
});

// --- Basic User Authentication Endpoints (Conceptual - NOT SECURE FOR PRODUCTION) ---
// **WARNING: Passwords are stored in plain text in memory for this demo!**

app.post('/api/register', (req, res) => {
    try {
        const { username, email, password } = req.body;

        // Basic validation
        if (!username || !email || !password) {
            return res.status(400).json({ message: 'All fields are required.' });
        }

        // Check if user already exists
        const existingUser = users.find(u => u.username === username || u.email === email);
        if (existingUser) {
            return res.status(409).json({ message: 'Username or email already exists.' });
        }

        const newUser = { username, email, password };
        users.push(newUser); // Add the new user to the in-memory array
        console.log('New user registered (in-memory):', newUser);
        res.status(201).json({ message: 'User registered successfully!' });
    } catch (error) {
        console.error('Registration failed:', error);
        res.status(500).json({ message: 'Registration failed.', error: error.message });
    }
});

app.post('/api/login', (req, res) => {
    try {
        const { username, password } = req.body;

        // Basic validation
        if (!username || !password) {
            return res.status(400).json({ message: 'Username and password are required.' });
        }

        const user = users.find(u => u.username === username && u.password === password); // **WARNING: Plain text password comparison!**
        if (!user) {
            return res.status(400).json({ message: 'Invalid username or password.' });
        }

        // In a real app, you'd generate a JWT token here and send it to the client
        res.json({ message: 'Logged in successfully!', username: user.username });
    } catch (error) {
        console.error('Login failed:', error);
        res.status(500).json({ message: 'Login failed.', error: error.message });
    }
});

// --- Start Server ---
app.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
  console.log(`Frontend accessible at http://localhost:${PORT}/index.html`);
  console.log(`Direct homepage access at http://localhost:${PORT}/homepage.html`);
  console.log(`Images will be saved in: ${uploadsDir}`);
  console.warn('WARNING: All data is stored in-memory and will be lost on server restart!');
});
