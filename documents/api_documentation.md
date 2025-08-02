# DigiMart API Documentation

This document provides comprehensive documentation for the DigiMart platform API, a digital marketplace for buying and selling digital products.

## Base Configuration

- **Base URL**: `http://localhost:5000/api`
- **Authentication**: JWT Bearer Token
- **Content-Type**: `application/json` (except for file uploads)
- **CORS**: Enabled for multiple origins

## Authentication

All authenticated endpoints require a JWT token in the Authorization header:
```
Authorization: Bearer <JWT_TOKEN>
```

## API Endpoints

### 🔐 Authentication Endpoints

#### 1. User Signup
**POST** `/auth/signup`

Register a new user (buyer or seller) on the platform using email and password.

**Request Body:**
```json
{
  "UserName": "John Doe",
  "UserEmail": "john@example.com",
  "UserPassword": "password123",
  "UserRole": "Buyer" // or "Seller"
}
```

**Response (201 Created):**
```json
{
  "success": true,
  "message": "User created successfully"
}
```

**Response (400 Bad Request):**
```json
{
  "success": false,
  "message": "User already exists"
}
```

#### 2. User Login
**POST** `/auth/login`

Authenticate a user and receive a JWT token.

**Request Body:**
```json
{
  "loginEMail": "john@example.com",
  "Password": "password123"
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "message": "User logged in successfully",
  "data": {
    "user": {
      "_id": "user_id",
      "UserName": "John Doe",
      "UserEmail": "john@example.com",
      "UserRole": "Buyer",
      "createdAt": "2024-01-15T10:00:00.000Z",
      "updatedAt": "2024-01-15T10:00:00.000Z"
    },
    "token": "jwt_token_here"
  }
}
```

#### 3. User Logout
**GET** `/auth/logout`

Log out the current user (token invalidation handled client-side).

**Response (200 OK):**
```json
{
  "success": true,
  "message": "User logged out successfully"
}
```



### 📧 OTP Verification Endpoints (Purchase Process)

#### 1. Send OTP for Purchase
**POST** `/auth/send-purchase-otp`

Send OTP to user's email before allowing purchase.

**Headers:** `Authorization: Bearer <JWT_TOKEN>`

**Request Body:**
```json
{
  "userEmail": "john@example.com"
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "message": "OTP sent successfully to your email"
}
```

#### 2. Verify OTP for Purchase
**POST** `/auth/verify-purchase-otp`

Verify OTP before allowing purchase process.

**Headers:** `Authorization: Bearer <JWT_TOKEN>`

**Request Body:**
```json
{
  "userEmail": "john@example.com",
  "otp": "123456"
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "message": "OTP verified successfully",
  "data": {
    "purchaseToken": "purchase_verification_token"
  }
}
```

**Response (400 Bad Request):**
```json
{
  "success": false,
  "message": "Invalid or expired OTP"
}
```

### 📦 Product Management Endpoints

#### 1. Get All Products
**GET** `/product/getAllProducts`

Retrieve all products from the platform with pagination support.

**Query Parameters:**
- `page`: Page number (default: 1, min: 1)
- `limit`: Number of items per page (default: 10, min: 1, max: 50)
- `sort`: Sort field (default: 'createdAt')
  - Available fields: 'createdAt', 'updatedAt', 'price', 'prodName'
- `order`: Sort order ('asc' or 'desc', default: 'desc')
- `category`: Filter by product category (optional)
  - Available categories: 'software', 'ebook', 'music', 'online courses', 'digital art'
- `search`: Search in product name and description (optional, case-insensitive)

**Example Requests:**
```
GET /product/getAllProducts
GET /product/getAllProducts?page=2&limit=20
GET /product/getAllProducts?page=1&limit=10&sort=price&order=asc
GET /product/getAllProducts?category=ebook&search=programming
```

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Products fetched successfully",
  "data": {
    "products": [
      {
        "_id": "product_id",
        "prodName": "Digital Art Pack",
        "ProdDiscription": "High-quality digital art collection",
        "ProdImage": ["image_url"],
        "price": 29.99,
        "sellerId": {
          "_id": "seller_id",
          "UserName": "Art Seller",
          "UserEmail": "seller@example.com"
        },
        "Prodcategory": "digital art",
        "createdAt": "2024-01-15T10:00:00.000Z",
        "updatedAt": "2024-01-15T10:00:00.000Z"
      }
    ],
    "pagination": {
      "currentPage": 1,
      "totalPages": 5,
      "totalItems": 50,
      "itemsPerPage": 10,
      "hasNextPage": true,
      "hasPrevPage": false,
      "nextPage": 2,
      "prevPage": null
    }
  }
}
```

#### 2. Get Products by Seller
**GET** `/product/seller/:sellerId`

Retrieve all products from a specific seller with pagination support.

**Parameters:**
- `sellerId`: MongoDB ObjectId of the seller

**Query Parameters:**
- `page`: Page number (default: 1, min: 1)
- `limit`: Number of items per page (default: 10, min: 1, max: 50)
- `sort`: Sort field (default: 'createdAt')
  - Available fields: 'createdAt', 'updatedAt', 'price', 'prodName'
- `order`: Sort order ('asc' or 'desc', default: 'desc')
- `search`: Search in product name and description (optional, case-insensitive)

**Example Requests:**
```
GET /product/seller/507f1f77bcf86cd799439011
GET /product/seller/507f1f77bcf86cd799439011?page=2&limit=20
GET /product/seller/507f1f77bcf86cd799439011?sort=price&order=asc
GET /product/seller/507f1f77bcf86cd799439011?search=digital&page=1&limit=10
```

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Seller products fetched successfully",
  "data": {
    "products": [...],
    "pagination": {
      "currentPage": 1,
      "totalPages": 3,
      "totalItems": 25,
      "itemsPerPage": 10,
      "hasNextPage": true,
      "hasPrevPage": false,
      "nextPage": 2,
      "prevPage": null
    }
  }
}
```

#### 3. Add Product (Sellers Only)
**POST** `/product/addProduct`

Add a new product to the platform with S3 file upload support.

**Headers:** 
- `Authorization: Bearer <JWT_TOKEN>` (Required for seller authentication)
- `Content-Type: multipart/form-data`

**Request Body (form-data):**
```
prodName: "My Digital Product"
ProdDiscription: "Detailed product description"
price: 29.99
sellerId: "seller_mongodb_id"
Prodcategory: "software" // Options: "ebook", "music", "online courses", "digital art"
file: [uploaded product file] // Required: Product file (PDF, ZIP, MP3)
thumbnail: [uploaded thumbnail image] // Optional: Product thumbnail image
```

**File Upload Requirements:**
- **Product File**: Required, supports various digital formats
  - E-books: `.pdf`, `.epub`, `.mobi`
  - Music: `.mp3`, `.wav`, `.flac`, `.aac`
  - Courses: `.zip`, `.mp4`, `.mov`
  - Digital Art: `.psd`, `.ai`, `.svg`, `.png`, `.jpg`
- **Thumbnail Image**: Optional, for product preview
  - Formats: `.jpg`, `.jpeg`, `.png`, `.webp`
  - Max size: 5MB
  - Recommended: 800x600px or 16:9 aspect ratio
- **File Size Limits**: 
  - Product files: Max 100MB
  - Thumbnail: Max 5MB

**Response (201 Created):**
```json
{
  "success": true,
  "message": "Product created successfully",
  "data": {
    "product": {
      "_id": "product_id",
      "prodName": "My Digital Product",
      "ProdDiscription": "Detailed product description",
      "ProdImage": ["s3_product_file_url"],
      "thumbnail": "s3_thumbnail_url",
      "price": 29.99,
      "sellerId": "seller_id",
      "Prodcategory": "ebook",
      "fileSize": "2.5MB",
      "downloadCount": 0,
      "createdAt": "2024-01-15T10:00:00.000Z",
      "updatedAt": "2024-01-15T10:00:00.000Z"
    },
    "uploadInfo": {
      "productFile": {
        "fileName": "my_software_v1.0.zip",
        "s3Key": "products/seller_id/product_id/my_software_v1.0.zip",
        "s3Url": "https://your-bucket.s3.amazonaws.com/products/seller_id/product_id/my_software_v1.0.zip",
        "fileSize": "2.5MB",
        "contentType": "application/zip"
      },
      "thumbnail": {
        "fileName": "thumbnail.jpg",
        "s3Key": "thumbnails/seller_id/product_id/thumbnail.jpg",
        "s3Url": "https://your-bucket.s3.amazonaws.com/thumbnails/seller_id/product_id/thumbnail.jpg",
        "fileSize": "150KB",
        "contentType": "image/jpeg"
      }
    }
  }
}
```

**Response (400 Bad Request):**
```json
{
  "success": false,
  "message": "File upload failed",
  "errors": {
    "file": "Invalid file format. Supported formats: zip, pdf, mp3, etc.",
    "size": "File size exceeds maximum limit of 100MB"
  }
}
```

**Response (401 Unauthorized):**
```json
{
  "success": false,
  "message": "Authentication required. Please login as a seller."
}
```

#### 4. Update Product
**PUT** `/product/updateProduct/:id`

Update an existing product.

**Parameters:**
- `id`: MongoDB ObjectId of the product

**Request Body:**
```json
{
  "prodName": "Updated Product Name",
  "ProdDiscription": "Updated description",
  "price": 39.99,
  "Prodcategory": "ebook"
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Product updated successfully",
  "data": {
    // Updated product object
  }
}
```

#### 5. Delete Product
**DELETE** `/product/deleteProduct/:id`

Delete a product from the platform.

**Parameters:**
- `id`: MongoDB ObjectId of the product

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Product deleted successfully"
}
```

### 🛒 Order Management Endpoints

#### 1. Create Order from Cart
**POST** `/order/create-from-cart`

Create a new order from cart items. Requires OTP verification for buyers.

**Headers:** `Authorization: Bearer <JWT_TOKEN>`

**Request Body:**
```json
{
  "cartItemIds": ["cart_item_id1", "cart_item_id2"], // Optional: Specific cart items to order
  "recipientEmail": "recipient@example.com", // Optional: Email for purchase recipient
  "purchaseToken": "purchase_verification_token" // Required for buyers
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Order created successfully",
  "data": {
    "order": {
      "_id": "order_id",
      "buyerId": "buyer_id",
      "recipientEmail": "recipient@example.com",
      "orderItems": [
        {
          "productId": "product_id",
          "productPrice": 29.99,
          "productName": "Digital Product"
        }
      ],
      "totalPrice": 29.99,
      "status": "Pending",
      "createdAt": "2024-01-15T10:00:00.000Z"
    },
    "cartUpdated": {
      "remainingItems": 2,
      "removedItems": ["cart_item_id1", "cart_item_id2"]
    }
  }
}
```

#### 2. Create Order (Direct)
**POST** `/order/create`

Create a new order for specific products. Requires OTP verification for buyers.

**Headers:** `Authorization: Bearer <JWT_TOKEN>`

**Request Body:**
```json
{
  "buyerId": "buyer_mongodb_id",
  "orderItems": [
    {
      "productId": "product_id",
      "productPrice": 29.99
    }
  ],
  "recipientEmail": "recipient@example.com", // Optional: Email for purchase recipient
  "totalPrice": 29.99,
  "status": "Pending", // Options: "Pending", "Rejected", "Success"
  "purchaseToken": "purchase_verification_token" // Required for buyers
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Order created successfully",
  "data": {
    "order": {
      "_id": "order_id",
      "buyerId": "buyer_id",
      "recipientEmail": "recipient@example.com",
      "orderItems": [...],
      "totalPrice": 29.99,
      "status": "Pending",
      "createdAt": "2024-01-15T10:00:00.000Z"
    }
  }
}
```

**Response (400 Bad Request):**
```json
{
  "success": false,
  "message": "Purchase verification required. Please verify OTP first."
}
```

#### 3. Get Order
**GET** `/order/get/:orderId`

Retrieve order details by order ID.

**Parameters:**
- `orderId`: MongoDB ObjectId of the order

**Response (200 OK):**
```json
{
  "success": true,
  "order": {
    "_id": "order_id",
    "buyerId": "buyer_id",
    "recipientEmail": "recipient@example.com",
    "orderItems": [
      {
        "productId": "product_id",
        "productPrice": 29.99,
        "productName": "Digital Product"
      }
    ],
    "totalPrice": 29.99,
    "status": "Pending",
    "createdAt": "2024-01-15T10:00:00.000Z",
    "updatedAt": "2024-01-15T10:00:00.000Z"
  }
}
```

#### 4. Update Order
**PUT** `/order/update/:orderId`

Update order status or details.

**Parameters:**
- `orderId`: MongoDB ObjectId of the order

**Request Body:**
```json
{
  "status": "Success",
  "totalPrice": 29.99,
  "recipientEmail": "new_recipient@example.com"
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Order updated successfully",
  "data": {
    "order": {
      "_id": "order_id",
      "status": "Success",
      "totalPrice": 29.99,
      "recipientEmail": "new_recipient@example.com",
      "updatedAt": "2024-01-15T10:00:00.000Z"
    }
  }
}
```

#### 5. Delete Order
**DELETE** `/order/delete/:orderId`

Delete an order from the system.

**Parameters:**
- `orderId`: MongoDB ObjectId of the order

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Order deleted successfully"
}
```

### ⭐ Review Management Endpoints

#### 1. Add Review
**POST** `/review/reviewAdd`

Add a review for a product.

**Request Body:**
```json
{
  "productId": "product_mongodb_id",
  "BuyerId": "buyer_mongodb_id",
  "Rating": 5, // 1-5 scale
  "comments": "Great product! Highly recommended."
}
```

**Response (200 OK):**
```json
{
  "data": "Review is added",
  "resp": {
    "_id": "review_id",
    "productId": "product_id",
    "BuyerId": "buyer_id",
    "Rating": 5,
    "comments": "Great product! Highly recommended.",
    "createdAt": "2024-01-15T10:00:00.000Z",
    "updatedAt": "2024-01-15T10:00:00.000Z"
  }
}
```

### 🛒 Cart Management Endpoints

#### Cart Lifecycle Overview

The cart system follows a smart lifecycle approach:
- **NO CART**: User has no cart until first "Add to Cart" action
- **CART CREATED**: Cart is created only when user first adds an item
- **CART EMPTIED**: Items are removed but cart shell is preserved
- **CART REFILLED**: New items are added to existing cart shell

#### 1. Add Product to Cart
**POST** `/cart/add`

Add a product to the user's shopping cart. Creates cart if it doesn't exist.

**Headers:** `Authorization: Bearer <JWT_TOKEN>`

**Request Body:**
```json
{
  "productId": "product_mongodb_id"
}
```

**Response (200 OK) - First Item (Cart Created):**
```json
{
  "success": true,
  "message": "Cart created and product added successfully",
  "data": {
    "cartCreated": true,
    "cartItem": {
      "_id": "cart_item_id",
      "userId": "user_id",
      "productId": {
        "_id": "product_id",
        "prodName": "Digital Product",
        "price": 29.99,
        "thumbnail": "s3_thumbnail_url"
      },
      "addedAt": "2024-01-15T10:00:00.000Z"
    },
    "cartSummary": {
      "totalItems": 1,
      "subtotal": 29.99,
      "tax": 2.99,
      "total": 32.98
    }
  }
}
```

**Response (200 OK) - Additional Items:**
```json
{
  "success": true,
  "message": "Product added to cart successfully",
  "data": {
    "cartCreated": false,
    "cartItem": {
      "_id": "cart_item_id",
      "userId": "user_id",
      "productId": {
        "_id": "product_id",
        "prodName": "Digital Product",
        "price": 29.99,
        "thumbnail": "s3_thumbnail_url"
      },
      "addedAt": "2024-01-15T10:00:00.000Z"
    },
    "cartSummary": {
      "totalItems": 3,
      "subtotal": 89.97,
      "tax": 8.99,
      "total": 98.96
    }
  }
}
```

#### 2. Get Cart Contents
**GET** `/cart/items`

Retrieve all items in the user's shopping cart.

**Headers:** `Authorization: Bearer <JWT_TOKEN>`

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Cart items retrieved successfully",
  "data": {
    "cartItems": [
      {
        "_id": "cart_item_id",
        "productId": {
          "_id": "product_id",
          "prodName": "Digital Product",
          "ProdDiscription": "Product description",
          "price": 29.99,
          "thumbnail": "s3_thumbnail_url",
          "sellerId": {
            "_id": "seller_id",
            "UserName": "Seller Name"
          }
        },
        "quantity": 1,
        "addedAt": "2024-01-15T10:00:00.000Z"
      }
    ],
    "cartSummary": {
      "totalItems": 3,
      "subtotal": 89.97,
      "tax": 8.99,
      "total": 98.96,
      "itemCount": 3
    }
  }
}
```

#### 3. Remove Item from Cart
**DELETE** `/cart/remove/:itemId`

Remove a specific item from the shopping cart.

**Parameters:**
- `itemId`: MongoDB ObjectId of the cart item

**Headers:** `Authorization: Bearer <JWT_TOKEN>`

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Item removed from cart successfully",
  "data": {
    "cartSummary": {
      "totalItems": 2,
      "subtotal": 59.98,
      "tax": 5.99,
      "total": 65.97
    }
  }
}
```

#### 4. Clear Entire Cart
**DELETE** `/cart/clear`

Remove all items from the shopping cart while preserving the cart shell.

**Headers:** `Authorization: Bearer <JWT_TOKEN>`

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Cart cleared successfully - cart shell preserved for future use",
  "data": {
    "cartPreserved": true,
    "cartSummary": {
      "totalItems": 0,
      "subtotal": 0,
      "tax": 0,
      "total": 0
    }
  }
}
```

**Response (404 Not Found):**
```json
{
  "success": false,
  "message": "No cart found for this user"
}
```

#### 5. Get Cart Summary
**GET** `/cart/summary`

Get a quick summary of the cart contents without full product details.

**Headers:** `Authorization: Bearer <JWT_TOKEN>`

**Response (200 OK) - Cart with Items:**
```json
{
  "success": true,
  "data": {
    "cartExists": true,
    "hasItems": true,
    "totalItems": 3,
    "subtotal": 89.97,
    "tax": 8.99,
    "total": 98.96,
    "itemCount": 3
  }
}
```

**Response (200 OK) - Empty Cart (Shell Preserved):**
```json
{
  "success": true,
  "data": {
    "cartExists": true,
    "hasItems": false,
    "totalItems": 0,
    "subtotal": 0,
    "tax": 0,
    "total": 0,
    "itemCount": 0
  }
}
```

**Response (404 Not Found) - No Cart:**
```json
{
  "success": true,
  "data": {
    "cartExists": false,
    "hasItems": false,
    "totalItems": 0,
    "subtotal": 0,
    "tax": 0,
    "total": 0,
    "itemCount": 0
  }
}
```

#### 6. Get Cart Status
**GET** `/cart/status`

Get the current cart status and lifecycle information.

**Headers:** `Authorization: Bearer <JWT_TOKEN>`

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "cartStatus": "active", // "active", "empty", "none"
    "cartExists": true,
    "hasItems": true,
    "createdAt": "2024-01-15T10:00:00.000Z",
    "lastModified": "2024-01-15T11:30:00.000Z",
    "itemCount": 3,
    "totalValue": 98.96
  }
}
```

### 📧 Address Management Endpoints

#### 1. Add Email Address
**POST** `/address/add-email`

Add a new email address to the user's address book with comprehensive tracking.

**Headers:** `Authorization: Bearer <JWT_TOKEN>`

**Request Body:**
```json
{
  "email": "recipient@example.com",
  "label": "Work Email", // Optional: Custom label for the email
  "isDefault": false, // Optional: Set as default recipient
  "description": "Office email for work purchases", // Optional: Additional description
  "tags": ["work", "office"] // Optional: Tags for organization
}
```

**Response (201 Created):**
```json
{
  "success": true,
  "message": "Email address added successfully",
  "data": {
    "address": {
      "_id": "address_id",
      "userId": "user_id",
      "email": "recipient@example.com",
      "label": "Work Email",
      "description": "Office email for work purchases",
      "tags": ["work", "office"],
      "isDefault": false,
      "isVerified": false,
      "verificationAttempts": 0,
      "lastVerificationSent": null,
      "purchaseCount": 0,
      "totalSpent": 0,
      "createdAt": "2024-01-15T10:00:00.000Z",
      "updatedAt": "2024-01-15T10:00:00.000Z"
    }
  }
}
```

**Response (400 Bad Request):**
```json
{
  "success": false,
  "message": "Email address already exists in your address book"
}
```

#### 2. Get All Email Addresses
**GET** `/address/emails`

Retrieve all email addresses in the user's address book with comprehensive tracking data.

**Headers:** `Authorization: Bearer <JWT_TOKEN>`

**Query Parameters:**
- `sort`: Sort field (default: 'createdAt')
  - Available fields: 'createdAt', 'updatedAt', 'purchaseCount', 'totalSpent', 'label'
- `order`: Sort order ('asc' or 'desc', default: 'desc')
- `filter`: Filter by status (optional)
  - Options: 'verified', 'unverified', 'default'
- `search`: Search in email, label, or description (optional)

**Example Requests:**
```
GET /address/emails
GET /address/emails?sort=purchaseCount&order=desc
GET /address/emails?filter=verified&search=work
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "addresses": [
      {
        "_id": "address_id",
        "email": "recipient@example.com",
        "label": "Work Email",
        "description": "Office email for work purchases",
        "tags": ["work", "office"],
        "isDefault": true,
        "isVerified": true,
        "verificationAttempts": 1,
        "lastVerificationSent": "2024-01-15T10:00:00.000Z",
        "verifiedAt": "2024-01-15T10:05:00.000Z",
        "purchaseCount": 5,
        "totalSpent": 149.95,
        "lastPurchaseDate": "2024-01-14T15:30:00.000Z",
        "createdAt": "2024-01-15T10:00:00.000Z",
        "updatedAt": "2024-01-15T10:05:00.000Z"
      }
    ],
    "summary": {
      "totalAddresses": 3,
      "verifiedAddresses": 2,
      "unverifiedAddresses": 1,
      "defaultAddress": {
        "_id": "address_id",
        "email": "recipient@example.com",
        "label": "Work Email"
      },
      "totalPurchases": 8,
      "totalSpent": 239.92
    }
  }
}
```

#### 3. Update Email Address
**PUT** `/address/update/:addressId`

Update an existing email address details while preserving tracking data.

**Parameters:**
- `addressId`: MongoDB ObjectId of the address

**Headers:** `Authorization: Bearer <JWT_TOKEN>`

**Request Body:**
```json
{
  "email": "updated@example.com",
  "label": "Updated Label",
  "description": "Updated description",
  "tags": ["updated", "tag"],
  "isDefault": true
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Email address updated successfully",
  "data": {
    "address": {
      "_id": "address_id",
      "email": "updated@example.com",
      "label": "Updated Label",
      "description": "Updated description",
      "tags": ["updated", "tag"],
      "isDefault": true,
      "isVerified": false,
      "verificationAttempts": 0,
      "purchaseCount": 0,
      "totalSpent": 0,
      "updatedAt": "2024-01-15T10:00:00.000Z"
    }
  }
}
```

**Response (400 Bad Request):**
```json
{
  "success": false,
  "message": "Email address already exists in your address book"
}
```

#### 4. Delete Email Address
**DELETE** `/address/delete/:addressId`

Remove an email address from the address book.

**Parameters:**
- `addressId`: MongoDB ObjectId of the address

**Headers:** `Authorization: Bearer <JWT_TOKEN>`

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Email address deleted successfully"
}
```

#### 5. Set Default Email Address
**PUT** `/address/set-default/:addressId`

Set an email address as the default recipient.

**Parameters:**
- `addressId`: MongoDB ObjectId of the address

**Headers:** `Authorization: Bearer <JWT_TOKEN>`

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Default email address updated successfully",
  "data": {
    "defaultAddress": {
      "_id": "address_id",
      "email": "default@example.com",
      "label": "Default Email"
    }
  }
}
```

#### 6. Verify Email Address
**POST** `/address/verify-email`

Send OTP to verify an email address in the address book.

**Headers:** `Authorization: Bearer <JWT_TOKEN>`

**Request Body:**
```json
{
  "addressId": "address_mongodb_id"
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "message": "OTP sent to email address for verification"
}
```

#### 7. Confirm Email Verification
**POST** `/address/confirm-verification`

Confirm email verification with OTP and update tracking data.

**Headers:** `Authorization: Bearer <JWT_TOKEN>`

**Request Body:**
```json
{
  "addressId": "address_mongodb_id",
  "otp": "123456"
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Email address verified successfully",
  "data": {
    "address": {
      "_id": "address_id",
      "email": "verified@example.com",
      "isVerified": true,
      "verificationAttempts": 1,
      "verifiedAt": "2024-01-15T10:00:00.000Z",
      "updatedAt": "2024-01-15T10:00:00.000Z"
    }
  }
}
```

**Response (400 Bad Request):**
```json
{
  "success": false,
  "message": "Invalid or expired OTP"
}
```

#### 8. Get Address Analytics
**GET** `/address/analytics`

Get comprehensive analytics for all addresses in the user's address book.

**Headers:** `Authorization: Bearer <JWT_TOKEN>`

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "overview": {
      "totalAddresses": 5,
      "verifiedAddresses": 3,
      "unverifiedAddresses": 2,
      "defaultAddress": {
        "_id": "address_id",
        "email": "default@example.com",
        "label": "Default Email"
      }
    },
    "purchaseStats": {
      "totalPurchases": 15,
      "totalSpent": 449.85,
      "averageOrderValue": 29.99,
      "mostUsedAddress": {
        "_id": "address_id",
        "email": "work@example.com",
        "purchaseCount": 8,
        "totalSpent": 239.92
      }
    },
    "recentActivity": {
      "lastAddedAddress": "2024-01-15T10:00:00.000Z",
      "lastVerifiedAddress": "2024-01-14T15:30:00.000Z",
      "lastPurchaseDate": "2024-01-14T15:30:00.000Z"
    },
    "addressBreakdown": [
      {
        "email": "work@example.com",
        "purchaseCount": 8,
        "totalSpent": 239.92,
        "lastPurchase": "2024-01-14T15:30:00.000Z",
        "isVerified": true
      }
    ]
  }
}
```

#### 9. Get Address by ID
**GET** `/address/:addressId`

Get detailed information about a specific address.

**Parameters:**
- `addressId`: MongoDB ObjectId of the address

**Headers:** `Authorization: Bearer <JWT_TOKEN>`

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "address": {
      "_id": "address_id",
      "userId": "user_id",
      "email": "recipient@example.com",
      "label": "Work Email",
      "description": "Office email for work purchases",
      "tags": ["work", "office"],
      "isDefault": true,
      "isVerified": true,
      "verificationAttempts": 1,
      "lastVerificationSent": "2024-01-15T10:00:00.000Z",
      "verifiedAt": "2024-01-15T10:05:00.000Z",
      "purchaseCount": 5,
      "totalSpent": 149.95,
      "lastPurchaseDate": "2024-01-14T15:30:00.000Z",
      "createdAt": "2024-01-15T10:00:00.000Z",
      "updatedAt": "2024-01-15T10:05:00.000Z"
    },
    "purchaseHistory": [
      {
        "orderId": "order_id",
        "productName": "Digital Product",
        "amount": 29.99,
        "purchaseDate": "2024-01-14T15:30:00.000Z"
      }
    ]
  }
}
```

#### 10. Bulk Address Operations
**POST** `/address/bulk-operations`

Perform bulk operations on multiple addresses.

**Headers:** `Authorization: Bearer <JWT_TOKEN>`

**Request Body:**
```json
{
  "operation": "delete", // Options: "delete", "verify", "setDefault"
  "addressIds": ["address_id1", "address_id2"]
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Bulk operation completed successfully",
  "data": {
    "operation": "delete",
    "processedCount": 2,
    "successCount": 2,
    "failedCount": 0,
    "results": [
      {
        "addressId": "address_id1",
        "status": "success",
        "message": "Address deleted successfully"
      }
    ]
  }
}
```

### 💳 Transaction Management Endpoints

#### 1. Process Payment
**POST** `/transections/payment`

Process a payment transaction. Requires prior OTP verification for buyers.

**Headers:** `Authorization: Bearer <JWT_TOKEN>`

**Request Body:**
```json
{
  "orderId": "order_mongodb_id",
  "payMethod": "Paypal", // Options: "Paypal", "UPI"
  "status": "Success", // Options: "Pending", "Failed", "Success"
  "purchaseToken": "purchase_verification_token" // Required for buyers
}
```

**Response (200 OK):**
```json
{
  "data": "transaction_id"
}
```



## Error Handling

The API uses standard HTTP status codes and returns error responses in the following format:

```json
{
  "success": false,
  "message": "Error description"
}
```

### Common Status Codes:
- `200 OK`: Request successful
- `201 Created`: Resource created successfully
- `400 Bad Request`: Invalid request data
- `401 Unauthorized`: Authentication required
- `403 Forbidden`: Access denied
- `404 Not Found`: Resource not found
- `500 Internal Server Error`: Server error

## File Upload & S3 Integration

The API supports comprehensive file upload functionality for digital products through the `/product/addProduct` endpoint. All files are securely stored in Amazon S3 for reliable, scalable, and fast cloud storage.

### S3 Storage Structure:
```
your-s3-bucket/
├── products/
│   └── {seller_id}/
│       └── {product_id}/
│           └── {product_file}
├── thumbnails/
    └── {seller_id}/
        └── {product_id}/
            └── {thumbnail_image}

```

### Upload Requirements:
- **Content-Type**: `multipart/form-data`
- **Authentication**: JWT Bearer token required for sellers
- **File Fields**: 
  - `file`: Main product file (required)
  - `thumbnail`: Product preview image (optional)
- **Storage**: Amazon S3 bucket with organized folder structure
- **File Access**: Direct S3 URLs with proper access controls

### Supported File Formats by Category:

#### E-books & Documents:
- **E-books**: `.pdf`, `.epub`, `.mobi`, `.azw3`
- **Documents**: `.docx`, `.txt`, `.rtf`
- **Presentations**: `.pptx`, `.key`

#### Audio & Music:
- **Audio**: `.mp3`, `.wav`, `.flac`, `.aac`, `.ogg`
- **High Quality**: `.m4a`, `.wma`, `.opus`

#### Video & Courses:
- **Video**: `.mp4`, `.mov`, `.avi`, `.mkv`, `.wmv`
- **Web**: `.webm`, `.flv`
- **Archives**: `.zip` (for course materials)

#### Digital Art & Graphics:
- **Raster**: `.png`, `.jpg`, `.jpeg`, `.gif`, `.webp`, `.tiff`
- **Vector**: `.svg`, `.ai`, `.eps`, `.pdf`
- **Design**: `.psd`, `.xcf`, `.sketch`

### File Size Limits:
- **Product Files**: Maximum 100MB per file
- **Thumbnail Images**: Maximum 5MB per image
- **Total Upload**: Maximum 200MB per product

### S3 Security Features:
- **Access Control**: Files are private by default
- **Signed URLs**: Temporary access URLs for downloads
- **CORS Configuration**: Properly configured for web access
- **Encryption**: Server-side encryption enabled
- **Versioning**: File versioning for backup and recovery

### Upload Process:
1. **Validation**: File format and size validation
2. **Processing**: File optimization and thumbnail generation
3. **Upload**: Secure upload to S3 with proper metadata
4. **Database**: S3 URLs and metadata saved to MongoDB
5. **Response**: Complete product data with S3 information

### Error Handling:
- **Invalid Format**: Returns specific format requirements
- **Size Exceeded**: Clear size limit information
- **Upload Failed**: Retry mechanism with progress tracking
- **S3 Errors**: Graceful fallback and error reporting

## Authentication Flow

1. **Register**: Use `/auth/signup` to create a new account with email and password
2. **Login**: Use `/auth/login` to get a JWT token
3. **Browse Products**: Access product endpoints with JWT token
4. **Purchase Process**: 
   - Send OTP using `/auth/send-purchase-otp`
   - Verify OTP using `/auth/verify-purchase-otp` to receive purchase token
   - Create order with purchase token
   - Process payment with purchase token
5. **Logout**: Use `/auth/logout` to end the session

## Role-Based Access

- **Buyers**: Can view products, create orders (with OTP verification), add reviews
- **Sellers**: Can manage products, view orders for their products (no OTP required)
- **Both**: Can update their profile, view their own data

## Purchase Verification Process

For buyers making purchases:
1. User must be logged in with valid JWT token
2. Before creating an order, user must:
   - Request OTP via `/auth/send-purchase-otp`
   - Verify OTP via `/auth/verify-purchase-otp` to receive purchase token
3. Use purchase token in order creation and payment processing
4. Purchase token has limited validity (recommended: 10 minutes)

## Pagination Guidelines

### Best Practices:
- **Default Limits**: Use reasonable defaults (page=1, limit=10)
- **Maximum Limits**: Enforce maximum limit of 50 items per page
- **Performance**: Use indexed fields for sorting ('createdAt', 'price')
- **Consistency**: Always include pagination info in responses
- **URL State**: Include pagination parameters in URLs for bookmarking

### Error Handling:
- **Invalid Page**: If page exceeds totalPages, return empty results
- **Invalid Limit**: If limit exceeds max (50), use maximum allowed
- **Invalid Sort**: If sort field doesn't exist, use default 'createdAt'
- **Negative Values**: Convert negative page/limit values to 1

### Example Error Response:
```json
{
  "success": false,
  "message": "Invalid pagination parameters",
  "errors": {
    "page": "Page number must be greater than 0",
    "limit": "Limit must be between 1 and 50"
  }
}
```

## Rate Limiting & Security

- CORS enabled for specified origins
- JWT tokens for authentication
- Password hashing with bcrypt
- OTP verification for purchase security
- Request logging with Pino
- Static file serving for uploads

## Environment Configuration

Required environment variables:
- `PORT`: Server port (default: 5000)
- `JWT_SECRET`: Secret key for JWT tokens
- `MONGODB_URI`: MongoDB connection string
- `FRONTEND_URL`: Frontend application URL
- `EMAIL_SERVICE`: Email service configuration for OTP

### AWS S3 Configuration:
```bash
# AWS Credentials
AWS_ACCESS_KEY_ID=your_aws_access_key
AWS_SECRET_ACCESS_KEY=your_aws_secret_key
AWS_REGION=us-east-1

# S3 Bucket Configuration
S3_BUCKET_NAME=your-digimart-bucket
S3_BUCKET_REGION=us-east-1

# Optional S3 Settings
S3_ENCRYPTION=true
S3_VERSIONING=true
S3_CORS_ENABLED=true
S3_MAX_FILE_SIZE=104857600  # 100MB in bytes
S3_THUMBNAIL_MAX_SIZE=5242880  # 5MB in bytes

# CDN Configuration (optional)
CLOUDFRONT_DISTRIBUTION_ID=your_cloudfront_distribution_id
CLOUDFRONT_DOMAIN=your_cloudfront_domain.cloudfront.net
```

### S3 Bucket Setup Requirements:
1. **Bucket Creation**: Create a dedicated S3 bucket for DigiMart
2. **CORS Configuration**: Enable CORS for web uploads
3. **Bucket Policy**: Configure proper access policies
4. **Versioning**: Enable versioning for file recovery
5. **Encryption**: Enable server-side encryption
6. **Lifecycle Rules**: Configure automatic cleanup of temporary files

### Example S3 CORS Configuration:
```json
[
  {
    "AllowedHeaders": ["*"],
    "AllowedMethods": ["GET", "POST", "PUT", "DELETE"],
    "AllowedOrigins": ["http://localhost:3000", "https://yourdomain.com"],
    "ExposeHeaders": ["ETag"],
    "MaxAgeSeconds": 3000
  }
]
``` 