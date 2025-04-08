# React Product Database Exercise with Real Database

This project demonstrates how to build a React application that displays and manages product data from a MongoDB database instead of json-server.

## Prerequisites

- Node.js (v14.0.0 or higher)
- npm (v6.0.0 or higher)
- MongoDB installed locally or access to a cloud MongoDB database
- Basic knowledge of React, JavaScript, and backend development

## Step-by-Step Development Guide

### 1. Project Setup

Start by creating a new React application:

```bash
npx create-react-app product-database
cd product-database
```

### 2. Install Frontend Dependencies

Install the required packages for React frontend:

```bash
npm install react-router-dom @mui/material @mui/icons-material @emotion/react @emotion/styled axios
```

### 3. Set Up the Backend

Create a backend directory structure:

```bash
mkdir -p backend/models backend/routes backend/controllers backend/config
```

Install backend dependencies:

```bash
cd backend
npm init -y
npm install express mongoose dotenv cors morgan bcrypt jsonwebtoken
npm install --save-dev nodemon
cd ..
```

### 4. MongoDB Setup

Create a database configuration file:

```bash
touch backend/config/db.js
```

Add the following content to `backend/config/db.js`:

```javascript
const mongoose = require('mongoose');

const connectDB = async () => {
  try {
    const conn = await mongoose.connect(process.env.MONGO_URI, {
      useNewUrlParser: true,
      useUnifiedTopology: true,
    });
    console.log(`MongoDB Connected: ${conn.connection.host}`);
  } catch (error) {
    console.error(`Error: ${error.message}`);
    process.exit(1);
  }
};

module.exports = connectDB;
```

Create a `.env` file in the backend directory:

```bash
touch backend/.env
```

Add your MongoDB connection string:

```
MONGO_URI=mongodb://localhost:27017/product_database
PORT=5000
JWT_SECRET=your_jwt_secret
```

### 5. Create MongoDB Product Model

Create `backend/models/productModel.js`:

```javascript
const mongoose = require('mongoose');

const productSchema = mongoose.Schema(
  {
    name: {
      type: String,
      required: true,
    },
    category: {
      type: String,
      required: true,
    },
    brand: {
      type: String,
      required: true,
    },
    price: {
      type: Number,
      required: true,
      default: 0,
    },
    inStock: {
      type: Boolean,
      required: true,
      default: false,
    },
    features: {
      type: [String],
      required: true,
    },
    rating: {
      type: Number,
      required: true,
      default: 0,
    },
  },
  {
    timestamps: true,
  }
);

const Product = mongoose.model('Product', productSchema);

module.exports = Product;
```

### 6. Create Controllers

Create `backend/controllers/productController.js`:

```javascript
const Product = require('../models/productModel');

// Get all products
const getProducts = async (req, res) => {
  try {
    const products = await Product.find({});
    res.json(products);
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
};

// Get product by ID
const getProductById = async (req, res) => {
  try {
    const product = await Product.findById(req.params.id);
    if (product) {
      res.json(product);
    } else {
      res.status(404).json({ message: 'Product not found' });
    }
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
};

// Get products by category
const getProductsByCategory = async (req, res) => {
  try {
    const products = await Product.find({ category: req.params.category });
    res.json(products);
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
};

// Get all unique categories
const getCategories = async (req, res) => {
  try {
    const categories = await Product.distinct('category');
    res.json(categories);
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
};

// Create new product
const createProduct = async (req, res) => {
  try {
    const product = new Product(req.body);
    const createdProduct = await product.save();
    res.status(201).json(createdProduct);
  } catch (error) {
    res.status(400).json({ message: error.message });
  }
};

// Update product
const updateProduct = async (req, res) => {
  try {
    const product = await Product.findById(req.params.id);
    if (product) {
      Object.assign(product, req.body);
      const updatedProduct = await product.save();
      res.json(updatedProduct);
    } else {
      res.status(404).json({ message: 'Product not found' });
    }
  } catch (error) {
    res.status(400).json({ message: error.message });
  }
};

// Delete product
const deleteProduct = async (req, res) => {
  try {
    const product = await Product.findById(req.params.id);
    if (product) {
      await Product.deleteOne({ _id: req.params.id }); // Use deleteOne instead of remove
      res.json({ message: 'Product removed' });
    } else {
      res.status(404).json({ message: 'Product not found' });
    }
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
};

module.exports = {
  getProducts,
  getProductById,
  getProductsByCategory,
  getCategories,
  createProduct,
  updateProduct,
  deleteProduct,
};
```

### 7. Create API Routes

Create `backend/routes/productRoutes.js`:

```javascript
const express = require('express');
const router = express.Router();
const {
  getProducts,
  getProductById,
  getProductsByCategory,
  getCategories,
  createProduct,
  updateProduct,
  deleteProduct,
} = require('../controllers/productController');

router.route('/').get(getProducts).post(createProduct);
router.route('/categories').get(getCategories);
router.route('/category/:category').get(getProductsByCategory);
router.route('/:id').get(getProductById).put(updateProduct).delete(deleteProduct);

module.exports = router;
```

### 8. Create Express Server

Create `backend/server.js`:

```javascript
const express = require('express');
const dotenv = require('dotenv');
const cors = require('cors');
const morgan = require('morgan');
const connectDB = require('./config/db');

// Load env vars
dotenv.config();

// Connect to database
connectDB();

// Route files
const productRoutes = require('./routes/productRoutes');

const app = express();

// Body parser
app.use(express.json());

// Enable CORS
app.use(cors());

// Dev logging middleware
if (process.env.NODE_ENV === 'development') {
  app.use(morgan('dev'));
}

// Mount routers
app.use('/api/products', productRoutes);

// Set port
const PORT = process.env.PORT || 5000;

// Start server
app.listen(PORT, () => console.log(`Server running on port ${PORT}`));

// Handle unhandled promise rejections
process.on('unhandledRejection', (err, promise) => {
  console.log(`Error: ${err.message}`);
  // Close server & exit process
  server.close(() => process.exit(1));
});
```

### 9. Update Package.json for Backend

Add these scripts to your backend/package.json:

```json
"scripts": {
  "start": "node server.js",
  "dev": "nodemon server.js",
  "seed": "node scripts/seed.js"
}
```

### 10. Create Database Seed Script

Create a seed script to populate your database with initial product data from the existing product.json file:

```bash
mkdir -p backend/scripts
touch backend/scripts/seed.js
```

Add the following to `backend/scripts/seed.js`:

```javascript
const mongoose = require('mongoose');
const dotenv = require('dotenv');
const fs = require('fs');
const path = require('path');
const Product = require('../models/productModel');

// Load env vars
dotenv.config();

// Connect to DB
mongoose.connect(process.env.MONGO_URI, {
  useNewUrlParser: true,
  useUnifiedTopology: true,
});

// Read product data from product.json
const importDataFromFile = async () => {
  try {
    // Read the product.json file from the project root
    const filePath = path.resolve(__dirname, '../../../product.json');
    const jsonData = fs.readFileSync(filePath, 'utf8');
    const { products } = JSON.parse(jsonData);
    
    console.log(`Found ${products.length} products in the product.json file`);
    
    // Clear existing products
    await Product.deleteMany({});
    
    // Insert products from file
    // Map the data to match our MongoDB schema
    const formattedProducts = products.map(product => ({
      // Use the existing id from product.json as _id for MongoDB
      _id: product.id,
      name: product.name,
      category: product.category,
      brand: product.brand,
      price: product.price,
      inStock: product.inStock,
      features: product.features,
      rating: product.rating
    }));
    
    await Product.insertMany(formattedProducts);
    console.log('Data imported successfully');
    process.exit();
  } catch (error) {
    console.error(`Error: ${error}`);
    process.exit(1);
  }
};

importDataFromFile();
```

### 11. Update Frontend Service Layer

Create `src/services/productService.js` with the updated backend URL:

```javascript
import axios from 'axios';

const API_URL = 'http://localhost:5000/api';

export const getAllProducts = async () => {
  try {
    const response = await axios.get(`${API_URL}/products`);
    return response.data;
  } catch (error) {
    throw new Error('Failed to fetch products: ' + error.message);
  }
};

export const getProductById = async (id) => {
  try {
    const response = await axios.get(`${API_URL}/products/${id}`);
    return response.data;
  } catch (error) {
    throw new Error(`Failed to fetch product with id ${id}: ${error.message}`);
  }
};

export const getProductsByCategory = async (category) => {
  try {
    const response = await axios.get(`${API_URL}/products/category/${category}`);
    return response.data;
  } catch (error) {
    throw new Error(`Failed to fetch products in category ${category}: ${error.message}`);
  }
};

export const getCategories = async () => {
  try {
    const response = await axios.get(`${API_URL}/products/categories`);
    return response.data;
  } catch (error) {
    throw new Error('Failed to fetch categories: ' + error.message);
  }
};

export const createProduct = async (productData) => {
  try {
    const response = await axios.post(`${API_URL}/products`, productData);
    return response.data;
  } catch (error) {
    throw new Error('Failed to create product: ' + error.message);
  }
};

export const updateProduct = async (id, productData) => {
  try {
    const response = await axios.put(`${API_URL}/products/${id}`, productData);
    return response.data;
  } catch (error) {
    throw new Error('Failed to update product: ' + error.message);
  }
};

export const deleteProduct = async (id) => {
  try {
    const response = await axios.delete(`${API_URL}/products/${id}`);
    return response.data;
  } catch (error) {
    throw new Error('Failed to delete product: ' + error.message);
  }
};
```

### 12. Verify JSON Structure

Make sure your database models and product.json structure match. Your product.json has this structure:

```json
{
  "products": [
    {
      "id": "prod001",
      "name": "Wireless Mouse",
      "category": "Electronics",
      "brand": "TechGear",
      "price": 24.99,
      "inStock": true,
      "features": ["Ergonomic design", "2.4GHz wireless", "Adjustable DPI"],
      "rating": 4.5
    },
    // more products...
  ]
}
```

This is important when importing the data into your database and when handling API responses in your frontend.

### 13. Run the Application

#### Start the backend server:

```bash
cd backend
npm run dev
```

#### Seed the database with data from product.json:

```bash
cd backend
npm run seed
```

#### Start the React frontend:

```bash
# In another terminal window from the project root
npm start
```

Visit `http://localhost:3000` in your browser to see the application.

## Project Structure

```
project-root/
  ├── backend/
  │   ├── config/
  │   │   └── db.js
  │   ├── controllers/
  │   │   └── productController.js
  │   ├── models/
  │   │   └── productModel.js
  │   ├── routes/
  │   │   └── productRoutes.js
  │   ├── scripts/
  │   │   └── seed.js
  │   ├── .env
  │   ├── package.json
  │   └── server.js
  ├── public/
  ├── src/
  │   ├── components/
  │   │   ├── layout/
  │   │   │   └── Header.js
  │   │   └── products/
  │   │       ├── ProductDetail.js
  │   │       ├── ProductItem.js
  │   │       └── ProductList.js
  │   ├── services/
  │   │   └── productService.js
  │   ├── App.js
  │   └── index.js
  ├── .env
  └── package.json
```

## Features Implemented

1. **Real Database Integration**: MongoDB for data persistence
2. **RESTful API**: Complete CRUD operations for products
3. **Product List View**: Display all products with filtering by category
4. **Product Detail View**: Show detailed information for a selected product
5. **Admin Features**: Create, update, and delete products

## Additional Development Tips

1. **Authentication**: Add user authentication with JWT
2. **Pagination**: Implement pagination for large product lists
3. **Error Handling**: Implement proper error handling for API requests
4. **Form Validation**: Add validation for product creation and update forms
5. **Loading States**: Show loading indicators during data fetching
6. **Deployment**: Instructions for deploying both frontend and backend

## Extending the Application

1. **User Authentication**: Add login/register functionality
2. **User Roles**: Admin and regular user roles with different permissions
3. **Shopping Cart**: Allow users to add products to cart
4. **Checkout Process**: Implement a checkout flow with shipping details
5. **Order History**: Track and display user orders

## Available Scripts

### Backend
- `npm start`: Runs the server in production mode
- `npm run dev`: Runs the server with nodemon for development
- `npm run seed`: Seeds the database with initial product data

### Frontend
- `npm start`: Runs the app in development mode
- `npm test`: Launches the test runner
- `npm run build`: Builds the app for production

## License

MIT

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.
