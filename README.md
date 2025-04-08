# React Product Database Exercise

This project demonstrates how to build a React application that displays and manages product data from a local JSON database.

## Prerequisites

- Node.js (v14.0.0 or higher)
- npm (v6.0.0 or higher)
- Basic knowledge of React and JavaScript

## Step-by-Step Development Guide

### 1. Project Setup

Start by creating a new React application:

```bash
npx create-react-app product-database
cd product-database
```

### 2. Install Dependencies

Install the required packages:

```bash
npm install react-router-dom @mui/material @mui/icons-material @emotion/react @emotion/styled json-server
```

### 3. Create the Database Structure

Create a folder for your database and add the product.json file:

```bash
mkdir -p src/db
```

Copy the `product.json` file from the project root to `src/db/product.json`, which contains the product data.
```

### 4. Set Up JSON Server

Add a script to your package.json file to run JSON server:

```bash
{
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test",
    "eject": "react-scripts eject",
    "server": "json-server --watch src/db/product.json --port 3001"
  }
}
```

Start the JSON server in a separate terminal:

```bash
npm run server
```

Your API will be available at `http://localhost:3001/products`.

### 5. Create Service Layer

Create a service to interact with the product data via the JSON server:

```bash
mkdir -p src/services
```

Create `src/services/productService.js`:

```javascript
const API_URL = 'http://localhost:3001';

export const getAllProducts = async () => {
  const response = await fetch(`${API_URL}/products`);
  if (!response.ok) {
    throw new Error('Failed to fetch products');
  }
  return response.json();
};

export const getProductById = async (id) => {
  const response = await fetch(`${API_URL}/products/${id}`);
  if (!response.ok) {
    throw new Error(`Failed to fetch product with id ${id}`);
  }
  return response.json();
};

export const getProductsByCategory = async (category) => {
  const response = await fetch(`${API_URL}/products?category=${category}`);
  if (!response.ok) {
    throw new Error(`Failed to fetch products in category ${category}`);
  }
  return response.json();
};

export const getCategories = async () => {
  const products = await getAllProducts();
  const categories = [...new Set(products.map(product => product.category))];
  return categories;
};
```

### 6. Create Components

Set up your component folders:

```bash
mkdir -p src/components/layout src/components/products
```

#### Header Component

Create `src/components/layout/Header.js` with navigation and category menu:

```javascript
import React, { useState, useEffect } from 'react';
import { Link as RouterLink } from 'react-router-dom';
import { AppBar, Toolbar, Typography, Container, Button, Box, Menu, MenuItem } from '@mui/material';
import StorefrontIcon from '@mui/icons-material/Storefront';
import { getCategories } from '../../services/productService';

// Create a header with navigation links and a dropdown for categories
```

#### Product List Component

Create `src/components/products/ProductList.js` to display all products or filtered by category:

```javascript
import React, { useState, useEffect } from 'react';
import { useParams } from 'react-router-dom';
import { Container, Grid, Typography, Box, CircularProgress } from '@mui/material';
import { getAllProducts, getProductsByCategory } from '../../services/productService';
import ProductItem from './ProductItem';

// Fetch and display products, with category filtering support
```

#### Product Item Component

Create `src/components/products/ProductItem.js` for individual product cards:

```javascript
import React from 'react';
import { Link as RouterLink } from 'react-router-dom';
import { Card, CardContent, CardActions, Typography, Button, Chip, Box, Rating } from '@mui/material';

// Display individual product cards with key information
```

#### Product Detail Component

Create `src/components/products/ProductDetail.js` for detailed product information:

```javascript
import React, { useState, useEffect } from 'react';
import { useParams, useNavigate } from 'react-router-dom';
import { Container, Typography, Box, Paper, Chip, List, ListItem, ListItemText, Button, Rating, Divider } from '@mui/material';
import { getProductById } from '../../services/productService';

// Show detailed information about a specific product
```

### 7. Set Up Routing

Update `src/App.js` to include routing:

```javascript
import React from 'react';
import { BrowserRouter as Router, Routes, Route } from 'react-router-dom';
import { CssBaseline, ThemeProvider, createTheme } from '@mui/material';
import Header from './components/layout/Header';
import ProductList from './components/products/ProductList';
import ProductDetail from './components/products/ProductDetail';

// Configure routing and theme
```

Update `src/index.js`:

```javascript
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';
import './index.css';

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```

### 8. Start Development Server

Run the application:

```bash
npm start
```

Visit `http://localhost:3000` in your browser to see the application.

## Features to Implement

1. **Product List View**: Display all products with filtering by category
2. **Product Detail View**: Show detailed information for a selected product
3. **Navigation**: Implement header with navigation links
4. **Responsive Design**: Ensure the app works on different screen sizes

## Product Data Structure

Each product in the database has the following structure:

```json
{
  "id": "prod001",
  "name": "Wireless Mouse",
  "category": "Electronics",
  "brand": "TechGear", 
  "price": 24.99,
  "inStock": true,
  "features": [
    "Ergonomic design",
    "2.4GHz wireless",
    "Adjustable DPI"
  ],
  "rating": 4.5
}
```

## Additional Development Tips

1. **Error Handling**: Implement proper error handling when fetching data
2. **Loading States**: Show loading indicators during data fetching
3. **Responsive Design**: Use Material UI's Grid system for responsive layouts
4. **State Management**: For larger applications, consider Redux or Context API

## Available Scripts

- `npm start`: Runs the app in development mode at [http://localhost:3000](http://localhost:3000)
- `npm test`: Launches the test runner
- `npm run build`: Builds the app for production

## License

MIT

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.
