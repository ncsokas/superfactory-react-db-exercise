# React Product Database with MongoDB Prompt

This prompt provides guidance for building a React application with MongoDB integration that manages product data. Use this as a reference when asking Copilot to implement specific parts of the exercise.

## Project Structure

The application should follow this structure:
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

## Backend Setup Instructions

### Initial Setup Commands

These commands will create a new React app, install dependencies, and set up the project structure:
- Create a new React application using `create-react-app`
- Install frontend dependencies: `react-router-dom`, Material UI packages (`@mui/material`, `@mui/icons-material`, `@emotion/react`, `@emotion/styled`) and `axios`
- Create backend directory structure with folders for models, routes, controllers, config, and scripts
- Initialize backend with npm and install necessary packages: `express`, `mongoose`, `dotenv`, `cors`, `morgan` and `nodemon` (as dev dependency)

### Database Configuration

Create a `db.js` file in the backend/config folder that:
- Imports mongoose
- Creates a connectDB function that connects to MongoDB using the MONGO_URI from environment variables
- Handles connection errors
- Exports the connectDB function

### Environment Variables

Create a `.env` file in the backend directory with:
- MongoDB connection string (MONGO_URI=mongodb://localhost:27017/product_database)
- Port number (PORT=5000)

### Product Model

Create a product schema in `productModel.js` with the following fields:
- _id (String, required) - Will store the product ID from product.json
- name (String, required)
- category (String, required)
- brand (String, required)
- price (Number, required, default: 0)
- inStock (Boolean, required, default: false)
- features (Array of Strings, required)
- rating (Number, required, default: 0)
- Include timestamps
- Export the model as 'Product'

### Product Controller

Create controller functions in `productController.js` for:
- getProducts - Fetch all products
- getProductById - Find product by ID from request parameters
- getProductsByCategory - Find products by category from request parameters
- getCategories - Get all unique product categories
- createProduct - Create a new product from request body
- updateProduct - Update existing product by ID
- deleteProduct - Remove product by ID
- Export all functions as named exports

### API Routes

Set up Express routes in `productRoutes.js`:
- Create a router using express.Router()
- Import all controller functions
- Define routes:
  - GET and POST to '/' for getting all products and creating products
  - GET to '/categories' for getting all categories
  - GET to '/category/:category' for products by category
  - GET, PUT, DELETE to '/:id' for product operations by ID
- Export the router

### Express Server

Set up the Express server in `server.js`:
- Import required packages and the database connection
- Load environment variables with dotenv
- Connect to database
- Import routes
- Configure middleware for JSON parsing, CORS, and logging
- Mount the product routes at '/api/products'
- Listen on the specified port
- Add error handling for unhandled promise rejections

### Backend Package.json scripts

Add these scripts to the backend package.json:
- "start": "node server.js"
- "dev": "nodemon server.js"
- "seed": "node scripts/seed.js"

### Database Seed Script

Create a script `seed.js` in backend/scripts that:
- Connects to MongoDB using environment variables
- Reads the product.json file from the project root
- Clears existing products from the database
- Maps the product data from JSON to match the MongoDB schema (converting 'id' to '_id')
- Inserts the formatted products into MongoDB
- Logs success and handles errors

## Frontend Integration

### Product Service

Create a `productService.js` in src/services that:
- Defines the API_URL constant pointing to the backend (http://localhost:5000/api)
- Exports async functions for each API operation:
  - getAllProducts - GET request to fetch all products
  - getProductById - GET request with ID parameter
  - getProductsByCategory - GET request with category parameter
  - getCategories - GET request for categories
  - createProduct - POST request with product data
  - updateProduct - PUT request with ID and product data
  - deleteProduct - DELETE request with ID
- Each function should include proper error handling

### React Components

#### Header Component

Create a simple header with:
- Material UI AppBar and Toolbar
- Typography component linked to the home page
- "Product Database" as the title

#### ProductList Component

Create a component that:
- Uses React hooks (useState, useEffect)
- Fetches and displays products from the API
- Fetches categories for filtering
- Allows filtering products by category
- Uses Material UI Grid, Card, Typography, Box, Chip components
- Shows loading and error states
- Displays product cards with name, brand, price, category, and stock status
- Links each card to the product detail page

#### ProductDetail Component

Create a component that:
- Fetches product details by ID from URL parameters
- Uses React Router hooks (useParams, useNavigate)
- Shows product information in a card layout
- Displays product features in a list
- Shows product rating using Material UI Rating component
- Includes edit and delete buttons
- Implements a delete confirmation dialog
- Handles loading, error, and not-found states

#### ProductForm Component

Create a form component that:
- Handles both create and edit modes via props (isEdit)
- Gets product ID from URL parameters
- Fetches product data when in edit mode
- Fetches categories for the dropdown
- Uses a form with Material UI components
- Manages form state with React hooks
- Handles different field types:
  - Text fields for name, brand
  - Select dropdown for category
  - Number field for price
  - Switch for in-stock status
  - Rating component
  - Feature management with add/remove capability
- Submits data to create or update API endpoints

#### App Component

Create the main App component that:
- Sets up Material UI theme
- Configures React Router
- Defines routes for:
  - Home page (ProductList)
  - Product detail page
  - Create product page
  - Edit product page
- Includes the Header component

## Run and Test Commands

### Running the Application

To run the application:
1. Start the backend server with `cd backend && npm run dev`
2. Seed the database with `cd backend && npm run seed`
3. Start the frontend with `npm start` from the root directory

### Verification Steps

Test the following functionality:
1. Backend server running on port 5000
2. Database seeded with products from product.json
3. Frontend displaying products correctly
4. Filtering by category
5. Viewing product details
6. Adding new products
7. Editing existing products
8. Deleting products

## Troubleshooting Tips

### MongoDB Connection Issues
- Ensure MongoDB is installed and running
- Verify the connection string in the .env file
- Check network permissions

### Frontend-Backend Connection
- Confirm the backend server is running
- Check CORS configuration
- Verify API_URL is correct

### Data Loading Problems
- Run the seed script
- Check database connection and name
- Test API endpoints with tools like Postman

## Extensions

Consider adding these features:
- User authentication
- Product image support
- Product reviews and ratings
- Admin dashboard
- Pagination for large product lists
