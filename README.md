# React Product Database Exercise with MongoDB Integration

This project demonstrates how to build a React application that displays and manages product data from a MongoDB database.

> **Note:** All code snippets in this README are also available in the [reactdb.prompt.md](/prompts/reactdb.prompt.md) file, which can be used as a reference when implementing this exercise.

You can either work with the Agent Mode to create the files for you using the reactdb.prompt.md file. You can also use the sections there to prompt the Edit or the Ask Mode to help you implement specific parts of the exercise at a time.

You may need to troubleshoot during implementation. Remember you can use the #terminalLastCommand or #terminalSelection variables in all modes to guide the Copilot Agent to help you fix the issue.

Also try and experiment using the different LLMs, as the output may be more or less accurate based on the model you are using.

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

Install the required packages for the React frontend:

```bash
npm install react-router-dom @mui/material @mui/icons-material @emotion/react @emotion/styled axios
```

### 3. Set Up the Backend

Create a backend directory structure:

```bash
mkdir -p backend/models backend/routes backend/controllers backend/config
```

Initialize the backend with npm and install dependencies:

```bash
cd backend
npm init -y
npm install express mongoose dotenv cors morgan
npm install --save-dev nodemon
cd ..
```

### 4. MongoDB Setup

Create a database configuration file:

```bash
touch backend/config/db.js
```

Add the following content to `backend/config/db.js`:

<details>
<summary>Solution</summary>

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
</details>

Create a `.env` file in the backend directory:

```bash
touch backend/.env
```

Add your MongoDB connection string:

```
MONGO_URI=mongodb://localhost:27017/product_database
PORT=5000
```

### 5. Create MongoDB Product Model

Create `backend/models/productModel.js`:

<details>
<summary>Solution</summary>

```javascript
const mongoose = require('mongoose');

const productSchema = mongoose.Schema(
  {
    _id: {
      type: String,
      required: true
    },
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
</details>

### 6. Create Controllers

Create `backend/controllers/productController.js`:

<details>
<summary>Solution</summary>

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
      await product.deleteOne();
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
</details>

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

<details>
<summary>Solution</summary>

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
const server = app.listen(PORT, () => console.log(`Server running on port ${PORT}`));

// Handle unhandled promise rejections
process.on('unhandledRejection', (err) => {
  console.log(`Error: ${err.message}`);
  // Close server & exit process
  server.close(() => process.exit(1));
});
```
</details>

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

<details>
<summary>Solution</summary>

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
const importData = async () => {
  try {
    // Read the product.json file from the project root
    const filePath = path.resolve(__dirname, '../../../product.json');
    const jsonData = fs.readFileSync(filePath, 'utf8');
    const { products } = JSON.parse(jsonData);
    
    console.log(`Found ${products.length} products in the product.json file`);
    
    // Clear existing products
    await Product.deleteMany({});
    
    // Insert products from file
    const formattedProducts = products.map(product => ({
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

importData();
```
</details>

### 11. Create Frontend Components

#### Create ProductService

Create a file `src/services/productService.js`:

<details>
<summary>Solution</summary>

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
</details>

#### Create Header Component

Create `src/components/layout/Header.js`:

<details>
<summary>Solution</summary>

```jsx
import React from 'react';
import { AppBar, Toolbar, Typography, Container } from '@mui/material';
import { Link } from 'react-router-dom';

const Header = () => {
  return (
    <AppBar position="static" color="primary" elevation={0}>
      <Container>
        <Toolbar disableGutters>
          <Typography
            variant="h6"
            component={Link}
            to="/"
            sx={{
              textDecoration: 'none',
              color: 'white',
              fontWeight: 700,
            }}
          >
            Product Database
          </Typography>
        </Toolbar>
      </Container>
    </AppBar>
  );
};

export default Header;
```
</details>

#### Create ProductList Component

Create `src/components/products/ProductList.js`:

<details>
<summary>Solution</summary>

```jsx
import React, { useEffect, useState } from 'react';
import { Link } from 'react-router-dom';
import {
  Container,
  Grid,
  Card,
  CardContent,
  Typography,
  Box,
  Chip,
  FormControl,
  InputLabel,
  Select,
  MenuItem,
  Button
} from '@mui/material';
import { getAllProducts, getCategories } from '../../services/productService';

const ProductList = () => {
  const [products, setProducts] = useState([]);
  const [categories, setCategories] = useState([]);
  const [selectedCategory, setSelectedCategory] = useState('');
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    fetchProducts();
    fetchCategories();
  }, []);

  const fetchProducts = async () => {
    try {
      setLoading(true);
      const data = await getAllProducts();
      setProducts(data);
      setError(null);
    } catch (err) {
      setError('Failed to load products');
      console.error(err);
    } finally {
      setLoading(false);
    }
  };

  const fetchCategories = async () => {
    try {
      const data = await getCategories();
      setCategories(data);
    } catch (err) {
      console.error('Error fetching categories:', err);
    }
  };

  const filterByCategory = (category) => {
    setSelectedCategory(category);
  };

  // Filter products by selected category
  const filteredProducts = selectedCategory
    ? products.filter((product) => product.category === selectedCategory)
    : products;

  if (loading) return <Typography>Loading products...</Typography>;
  if (error) return <Typography color="error">{error}</Typography>;

  return (
    <Container maxWidth="lg" sx={{ mt: 4, mb: 4 }}>
      <Box sx={{ display: 'flex', justifyContent: 'space-between', mb: 4 }}>
        <Typography variant="h4" component="h1" gutterBottom>
          Products
        </Typography>
        <Box>
          <FormControl variant="outlined" sx={{ minWidth: 120, mr: 2 }}>
            <InputLabel>Category</InputLabel>
            <Select
              value={selectedCategory}
              onChange={(e) => filterByCategory(e.target.value)}
              label="Category"
            >
              <MenuItem value="">
                <em>All</em>
              </MenuItem>
              {categories.map((category) => (
                <MenuItem key={category} value={category}>
                  {category}
                </MenuItem>
              ))}
            </Select>
          </FormControl>
          <Button variant="contained" component={Link} to="/add-product">
            Add Product
          </Button>
        </Box>
      </Box>

      <Grid container spacing={3}>
        {filteredProducts.map((product) => (
          <Grid item key={product._id} xs={12} sm={6} md={4}>
            <Card 
              component={Link} 
              to={`/products/${product._id}`}
              sx={{ 
                height: '100%', 
                display: 'flex', 
                flexDirection: 'column',
                textDecoration: 'none',
                transition: '0.3s',
                '&:hover': {
                  transform: 'translateY(-5px)',
                  boxShadow: 3,
                },
              }}
            >
              <CardContent sx={{ flexGrow: 1 }}>
                <Typography variant="h6" component="h2" gutterBottom>
                  {product.name}
                </Typography>
                <Typography variant="body2" color="text.secondary" gutterBottom>
                  {product.brand}
                </Typography>
                <Typography variant="h6" color="primary">
                  ${product.price.toFixed(2)}
                </Typography>
                <Box sx={{ mt: 2, display: 'flex', justifyContent: 'space-between', alignItems: 'center' }}>
                  <Chip 
                    label={product.category} 
                    size="small" 
                    color="secondary" 
                  />
                  <Chip 
                    label={product.inStock ? 'In Stock' : 'Out of Stock'} 
                    size="small" 
                    color={product.inStock ? 'success' : 'error'} 
                  />
                </Box>
              </CardContent>
            </Card>
          </Grid>
        ))}
      </Grid>
    </Container>
  );
};

export default ProductList;
```
</details>

#### Create ProductDetail Component

Create `src/components/products/ProductDetail.js`:

<details>
<summary>Solution</summary>

```jsx
import React, { useEffect, useState } from 'react';
import { useParams, Link, useNavigate } from 'react-router-dom';
import {
  Container,
  Typography,
  Box,
  Chip,
  Button,
  Card,
  CardContent,
  List,
  ListItem,
  ListItemText,
  Rating,
  Dialog,
  DialogActions,
  DialogContent,
  DialogContentText,
  DialogTitle,
} from '@mui/material';
import ArrowBackIcon from '@mui/icons-material/ArrowBack';
import EditIcon from '@mui/icons-material/Edit';
import DeleteIcon from '@mui/icons-material/Delete';
import { getProductById, deleteProduct } from '../../services/productService';

const ProductDetail = () => {
  const { id } = useParams();
  const navigate = useNavigate();
  const [product, setProduct] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  const [openDialog, setOpenDialog] = useState(false);

  useEffect(() => {
    const fetchProduct = async () => {
      try {
        setLoading(true);
        const data = await getProductById(id);
        setProduct(data);
        setError(null);
      } catch (err) {
        setError('Failed to load product details');
        console.error(err);
      } finally {
        setLoading(false);
      }
    };

    fetchProduct();
  }, [id]);

  const handleDelete = async () => {
    try {
      await deleteProduct(id);
      navigate('/');
    } catch (err) {
      setError('Failed to delete product');
      console.error(err);
    }
    setOpenDialog(false);
  };

  if (loading) return <Typography>Loading product details...</Typography>;
  if (error) return <Typography color="error">{error}</Typography>;
  if (!product) return <Typography>Product not found</Typography>;

  return (
    <Container maxWidth="md" sx={{ mt: 4, mb: 4 }}>
      <Box sx={{ mb: 4 }}>
        <Button
          component={Link}
          to="/"
          startIcon={<ArrowBackIcon />}
          sx={{ mb: 2 }}
        >
          Back to Products
        </Button>
        
        <Box sx={{ display: 'flex', justifyContent: 'space-between', alignItems: 'center', mb: 3 }}>
          <Typography variant="h4" component="h1">
            {product.name}
          </Typography>
          
          <Box>
            <Button
              variant="outlined"
              startIcon={<EditIcon />}
              component={Link}
              to={`/edit-product/${product._id}`}
              sx={{ mr: 1 }}
            >
              Edit
            </Button>
            <Button
              variant="outlined"
              color="error"
              startIcon={<DeleteIcon />}
              onClick={() => setOpenDialog(true)}
            >
              Delete
            </Button>
          </Box>
        </Box>
        
        <Card elevation={2}>
          <CardContent>
            <Box sx={{ display: 'flex', justifyContent: 'space-between', mb: 2 }}>
              <Box>
                <Typography variant="subtitle1" color="text.secondary">
                  Brand
                </Typography>
                <Typography variant="body1">
                  {product.brand}
                </Typography>
              </Box>
              
              <Box>
                <Typography variant="subtitle1" color="text.secondary">
                  Category
                </Typography>
                <Chip label={product.category} color="secondary" />
              </Box>
              
              <Box>
                <Typography variant="subtitle1" color="text.secondary">
                  Availability
                </Typography>
                <Chip 
                  label={product.inStock ? 'In Stock' : 'Out of Stock'} 
                  color={product.inStock ? 'success' : 'error'} 
                />
              </Box>
              
              <Box>
                <Typography variant="subtitle1" color="text.secondary">
                  Price
                </Typography>
                <Typography variant="h6" color="primary">
                  ${product.price.toFixed(2)}
                </Typography>
              </Box>
            </Box>
            
            <Box sx={{ my: 3 }}>
              <Typography variant="subtitle1" color="text.secondary">
                Rating
              </Typography>
              <Box sx={{ display: 'flex', alignItems: 'center' }}>
                <Rating value={product.rating} precision={0.1} readOnly />
                <Typography variant="body2" sx={{ ml: 1 }}>
                  ({product.rating})
                </Typography>
              </Box>
            </Box>
            
            <Box>
              <Typography variant="subtitle1" color="text.secondary">
                Features
              </Typography>
              <List dense>
                {product.features.map((feature, index) => (
                  <ListItem key={index}>
                    <ListItemText primary={feature} />
                  </ListItem>
                ))}
              </List>
            </Box>
          </CardContent>
        </Card>
      </Box>

      <Dialog
        open={openDialog}
        onClose={() => setOpenDialog(false)}
      >
        <DialogTitle>Delete Product</DialogTitle>
        <DialogContent>
          <DialogContentText>
            Are you sure you want to delete {product.name}? This action cannot be undone.
          </DialogContentText>
        </DialogContent>
        <DialogActions>
          <Button onClick={() => setOpenDialog(false)}>Cancel</Button>
          <Button onClick={handleDelete} color="error">Delete</Button>
        </DialogActions>
      </Dialog>
    </Container>
  );
};

export default ProductDetail;
```
</details>

#### Create ProductForm Component

Create `src/components/products/ProductForm.js`:

<details>
<summary>Solution</summary>

```jsx
import React, { useState, useEffect } from 'react';
import { useParams, useNavigate } from 'react-router-dom';
import {
  Container,
  Typography,
  Box,
  TextField,
  Button,
  FormControlLabel,
  Switch,
  FormControl,
  InputLabel,
  Select,
  MenuItem,
  Chip,
  Stack,
  Rating
} from '@mui/material';
import { 
  getProductById, 
  createProduct, 
  updateProduct, 
  getCategories 
} from '../../services/productService';

const ProductForm = ({ isEdit = false }) => {
  const { id } = useParams();
  const navigate = useNavigate();
  const [loading, setLoading] = useState(isEdit);
  const [categories, setCategories] = useState([]);
  const [newFeature, setNewFeature] = useState('');
  
  const [formData, setFormData] = useState({
    name: '',
    category: '',
    brand: '',
    price: '',
    inStock: true,
    features: [],
    rating: 0
  });
  
  useEffect(() => {
    const loadCategories = async () => {
      try {
        const data = await getCategories();
        setCategories(data);
      } catch (err) {
        console.error('Error loading categories:', err);
      }
    };
    
    loadCategories();
    
    if (isEdit && id) {
      const fetchProduct = async () => {
        try {
          const product = await getProductById(id);
          setFormData({
            name: product.name,
            category: product.category,
            brand: product.brand,
            price: product.price,
            inStock: product.inStock,
            features: product.features,
            rating: product.rating
          });
          setLoading(false);
        } catch (err) {
          console.error('Error loading product:', err);
        }
      };
      fetchProduct();
    }
  }, [isEdit, id]);

  const handleChange = (e) => {
    const { name, value, checked, type } = e.target;
    setFormData({
      ...formData,
      [name]: type === 'checkbox' ? checked : value
    });
  };

  const handleRatingChange = (event, newValue) => {
    setFormData({
      ...formData,
      rating: newValue
    });
  };

  const handleAddFeature = () => {
    if (newFeature.trim() !== '') {
      setFormData({
        ...formData,
        features: [...formData.features, newFeature.trim()]
      });
      setNewFeature('');
    }
  };

  const handleRemoveFeature = (index) => {
    const updatedFeatures = [...formData.features];
    updatedFeatures.splice(index, 1);
    setFormData({
      ...formData,
      features: updatedFeatures
    });
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    try {
      // Convert price to number
      const productData = {
        ...formData,
        price: parseFloat(formData.price)
      };
      
      if (isEdit) {
        await updateProduct(id, productData);
      } else {
        await createProduct(productData);
      }
      navigate('/');
    } catch (err) {
      console.error('Error submitting form:', err);
    }
  };

  if (loading) {
    return <Typography>Loading product data...</Typography>;
  }

  return (
    <Container maxWidth="md" sx={{ mt: 4, mb: 4 }}>
      <Typography variant="h4" component="h1" gutterBottom>
        {isEdit ? 'Edit Product' : 'Add New Product'}
      </Typography>
      
      <Box component="form" onSubmit={handleSubmit} noValidate sx={{ mt: 2 }}>
        <TextField
          margin="normal"
          required
          fullWidth
          label="Product Name"
          name="name"
          value={formData.name}
          onChange={handleChange}
        />
        
        <Box sx={{ display: 'flex', gap: 2, my: 2 }}>
          <FormControl fullWidth margin="normal">
            <InputLabel>Category</InputLabel>
            <Select
              name="category"
              value={formData.category}
              label="Category"
              onChange={handleChange}
            >
              {categories.map((category) => (
                <MenuItem key={category} value={category}>{category}</MenuItem>
              ))}
              <MenuItem value="Other">Other</MenuItem>
            </Select>
          </FormControl>
          
          <TextField
            margin="normal"
            required
            fullWidth
            label="Brand"
            name="brand"
            value={formData.brand}
            onChange={handleChange}
          />
        </Box>
        
        <Box sx={{ display: 'flex', gap: 2, my: 2, alignItems: 'center' }}>
          <TextField
            margin="normal"
            required
            fullWidth
            label="Price"
            name="price"
            type="number"
            inputProps={{ step: "0.01", min: "0" }}
            value={formData.price}
            onChange={handleChange}
          />
          
          <FormControlLabel
            control={
              <Switch
                checked={formData.inStock}
                onChange={handleChange}
                name="inStock"
                color="primary"
              />
            }
            label="In Stock"
          />
        </Box>
        
        <Box sx={{ my: 3 }}>
          <Typography component="legend">Rating</Typography>
          <Rating
            name="rating"
            value={formData.rating}
            precision={0.5}
            onChange={handleRatingChange}
          />
        </Box>
        
        <Box sx={{ mt: 3 }}>
          <Typography variant="subtitle1" gutterBottom>
            Features
          </Typography>
          <Box sx={{ display: 'flex', mb: 2 }}>
            <TextField
              fullWidth
              label="Add Feature"
              value={newFeature}
              onChange={(e) => setNewFeature(e.target.value)}
              onKeyPress={(e) => {
                if (e.key === 'Enter') {
                  e.preventDefault();
                  handleAddFeature();
                }
              }}
            />
            <Button 
              variant="contained" 
              onClick={handleAddFeature} 
              sx={{ ml: 1 }}
            >
              Add
            </Button>
          </Box>
          
          <Stack direction="row" spacing={1} flexWrap="wrap" useFlexGap>
            {formData.features.map((feature, index) => (
              <Chip
                key={index}
                label={feature}
                onDelete={() => handleRemoveFeature(index)}
                sx={{ mt: 1 }}
              />
            ))}
          </Stack>
        </Box>
        
        <Box sx={{ mt: 4, mb: 2 }}>
          <Button
            type="submit"
            variant="contained"
            color="primary"
            size="large"
          >
            {isEdit ? 'Update Product' : 'Create Product'}
          </Button>
          <Button
            variant="outlined"
            onClick={() => navigate('/')}
            sx={{ ml: 2 }}
          >
            Cancel
          </Button>
        </Box>
      </Box>
    </Container>
  );
};

export default ProductForm;
```
</details>

#### Update App.js

Create or update your `src/App.js`:

<details>
<summary>Solution</summary>

```jsx
import React from 'react';
import { BrowserRouter as Router, Routes, Route } from 'react-router-dom';
import { CssBaseline, ThemeProvider, createTheme } from '@mui/material';
import Header from './components/layout/Header';
import ProductList from './components/products/ProductList';
import ProductDetail from './components/products/ProductDetail';
import ProductForm from './components/products/ProductForm';

// Create a theme instance
const theme = createTheme({
  palette: {
    primary: {
      main: '#1976d2',
    },
    secondary: {
      main: '#f50057',
    },
  },
});

function App() {
  return (
    <ThemeProvider theme={theme}>
      <CssBaseline />
      <Router>
        <Header />
        <Routes>
          <Route path="/" element={<ProductList />} />
          <Route path="/products/:id" element={<ProductDetail />} />
          <Route path="/add-product" element={<ProductForm isEdit={false} />} />
          <Route path="/edit-product/:id" element={<ProductForm isEdit={true} />} />
        </Routes>
      </Router>
    </ThemeProvider>
  );
}

export default App;
```
</details>

### 12. Run the Application

#### Start the backend:

```bash
cd backend
npm run dev
```

#### Seed the database with data from product.json:

```bash
cd backend
npm run seed
```

#### Start the frontend:

```bash
# In the root directory
npm start
```

Your React application with MongoDB integration should now be running. Visit `http://localhost:3000` to see the product list and perform CRUD operations.

## Validation Steps

1. Verify the backend is running on port 5000
2. Check that the database was seeded with products from product.json
3. Verify the frontend is displaying products correctly
4. Test creating, updating, and deleting products
5. Confirm that filtering by category works

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
  │   │       ├── ProductList.js
  │   │       └── ProductForm.js
  │   ├── services/
  │   │   └── productService.js
  │   ├── App.js
  │   └── index.js
  ├── .env
  └── package.json
```

## License

MIT

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.
