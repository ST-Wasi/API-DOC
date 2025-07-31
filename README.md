# Smart Connections Backend API Documentation

## Table of Contents
- [Authentication](#authentication)
- [Base URL](#base-url)
- [Headers](#headers)
- [API Endpoints](#api-endpoints)
  - [Authentication Routes](#authentication-routes)
  - [User Routes](#user-routes)
  - [Card Routes](#card-routes)
  - [Order Routes](#order-routes)
  - [Profile Routes](#profile-routes)
  - [Category Routes](#category-routes)
  - [Tag Routes](#tag-routes)
  - [Artist Routes](#artist-routes)
  - [Settings Routes](#settings-routes)

## Authentication

The API uses JWT (JSON Web Token) for authentication. Most endpoints require authentication except for public endpoints.

### Token Format
```
Bearer <your-jwt-token>
```

## Base URL
```
http://localhost:5000/api
```

## Headers

### Required Headers for Authenticated Routes
```
Authorization: Bearer <jwt-token>
Content-Type: application/json
```

### For File Upload Routes
```
Authorization: Bearer <jwt-token>
Content-Type: multipart/form-data
```

---

## API Endpoints

### Authentication Routes

#### 1. Super Admin Login
- **URL:** `POST /auth/superadmin/login`
- **Description:** Login for super admin users
- **Authentication:** Not required
- **Request Body:**
```json
{
  "email": "superadmin@gmail.com",
  "password": "123456789"
}
```
- **Response:**
```json
{
  "success": true,
  "token": "jwt-token-here",
  "superAdmin": {
    "_id": "user-id",
    "email": "superadmin@gmail.com",
    "name": "super admin",
    "role": "superAdmin"
  }
}
```

---

### User Routes

#### 1. User Login (OTP Generation)
- **URL:** `POST /user/login`
- **Description:** Generate OTP for user login
- **Authentication:** Not required
- **Request Body:**
```json
{
  "email": "user@example.com",
  "name": "John Doe",
  "phone": "+1234567890"
}
```
- **Response:**
```json
{
  "_id": "user-id",
  "name": "John Doe",
  "email": "user@example.com",
  "phone": "+1234567890",
  "role": "user",
  "isVerified": false,
  "cards": []
}
```

#### 2. Verify OTP
- **URL:** `POST /user/verify-otp`
- **Description:** Verify OTP and get authentication token
- **Authentication:** Not required
- **Request Body:**
```json
{
  "email": "user@example.com",
  "otp": "1234"
}
```
- **Response:**
```json
{
  "token": "jwt-token-here"
}
```

#### 3. Get Card Profile Info by Username
- **URL:** `GET /user/get-user/username/:username`
- **Description:** Get profile information by unique username
- **Authentication:** Not required
- **Response:**
```json
{
  "data": {
    "_id": "profile-id",
    "uniqueId": "username",
    "name": "John Doe",
    "designation": "Developer",
    "companyName": "Tech Corp",
    "logo": "logo-filename.jpg",
    "orderId": {
      "_id": "order-id",
      "product": {
        "_id": "card-id",
        "productTitle": "Business Card",
        "price": 29.99
      }
    }
  }
}
```

---

### Card Routes

#### 1. Get All Cards (Public)
- **URL:** `GET /card/get-all-cards`
- **Description:** Get all published cards with pagination and optional filtering
- **Authentication:** Not required
- **Query Parameters:**
  - `page` (optional): Page number (default: 1)
  - `limit` (optional): Items per page (default: 10)
  - `artistId` (optional): Filter cards by specific artist ID - returns all cards created by that artist
  - `category` (optional): Filter cards by specific category ID - returns all cards belonging to that category
- **Filtering Examples:**
  - `GET /card/get-all-cards` - Returns all cards
  - `GET /card/get-all-cards?artistId=artist123` - Returns all cards by artist with ID "artist123"
  - `GET /card/get-all-cards?category=category456` - Returns all cards in category with ID "category456"
  - `GET /card/get-all-cards?artistId=artist123&category=category456` - Returns cards by specific artist in specific category
- **Response:**
```json
{
  "data": [
    {
      "_id": "card-id",
      "productTitle": "Business Card",
      "productSubtitle": "Professional Design",
      "price": 29.99,
      "discountedPrice": 24.99,
      "stock": 100,
      "artist": {
        "_id": "artist-id",
        "name": "Artist Name"
      },
      "categories": [
        {
          "_id": "category-id",
          "name": "Business"
        }
      ]
    }
  ],
  "limit": 10,
  "page": 1,
  "total": 50
}
```

#### 2. Get Cards by Category
- **URL:** `GET /card/by-category/:id`
- **Description:** Get all cards for a specific category
- **Authentication:** Not required
- **Response:**
```json
[
  {
    "_id": "card-id",
    "productTitle": "Business Card",
    "price": 29.99,
    "category": "category-id"
  }
]
```

#### 3. Create New Card
- **URL:** `POST /card/create-card`
- **Description:** Create a new card (Admin/Any role)
- **Authentication:** Required
- **Content-Type:** `multipart/form-data`
- **Request Body:**
```form-data
productTitle: "Business Card"
productSubtitle: "Professional Design"
productCustomUrl: "business-card-001"
price: "29.99"
tax: "2.99"
discountedPrice: "24.99"
stock: "100"
category: "category-id"
categories: ["category-id-1", "category-id-2"]
artist: "artist-id"
features: "Feature 1, Feature 2"
productFaq: "[{\"question\":\"FAQ 1\",\"answer\":\"Answer 1\"}]"
relatedProduct: "[\"card-id-1\", \"card-id-2\"]"
overviewText: "Card overview text"
tags: "[\"business\", \"professional\"]"
options: "[{\"name\":\"Color\",\"values\":[\"Red\",\"Blue\"]}]"
variants: "[{\"name\":\"Red Variant\",\"price\":29.99}]"
file: [image file]
```
- **Response:**
```json
{
  "message": "Card created successfully"
}
```

#### 4. Update Card
- **URL:** `PUT /card/update-card/:cardId`
- **Description:** Update card information (Admin only)
- **Authentication:** Required
- **Content-Type:** `multipart/form-data`
- **Request Body:**
```form-data
price: "35.99"
type: "premium"
file: [image file]
```
- **Response:**
```json
{
  "message": "Card updated successfully"
}
```

#### 5. Delete Card
- **URL:** `DELETE /card/delete-card/:cardId`
- **Description:** Delete a card (Admin only)
- **Authentication:** Required
- **Response:**
```json
{
  "message": "Card deleted successfully"
}
```

#### 6. Get User's Purchased Cards
- **URL:** `GET /card/get-cards-purchased`
- **Description:** Get cards purchased by authenticated user
- **Authentication:** Required
- **Query Parameters:**
  - `page` (optional): Page number (default: 1)
  - `limit` (optional): Items per page (default: 10)
- **Response:**
```json
{
  "data": [
    {
      "_id": "order-id",
      "product": {
        "_id": "card-id",
        "productTitle": "Business Card",
        "price": 29.99
      },
      "paymentStatus": "done"
    }
  ],
  "limit": 10,
  "page": 1,
  "total": 5
}
```

#### 7. Get Card by ID (Public)
- **URL:** `GET /card/get-card-by-id/:cardId`
- **Description:** Get card details by custom URL
- **Authentication:** Not required
- **Response:**
```json
{
  "card": {
    "_id": "card-id",
    "productTitle": "Business Card",
    "price": 29.99,
    "category": {
      "_id": "category-id",
      "name": "Business"
    }
  },
  "shippingCharges": 5.99
}
```

#### 8. Get Card by ID for Admin
- **URL:** `GET /card/get-card-by-id-for-admin/:cardId`
- **Description:** Get card details by ID for admin
- **Authentication:** Required (Admin)
- **Response:**
```json
{
  "card": {
    "_id": "card-id",
    "productTitle": "Business Card",
    "price": 29.99,
    "category": {
      "_id": "category-id",
      "name": "Business"
    }
  },
  "shippingCharges": 5.99
}
```

#### 9. Get All Cards for Admin
- **URL:** `GET /card/get-all-cards-for-admin`
- **Description:** Get all cards for admin panel
- **Authentication:** Required (Any role)
- **Query Parameters:**
  - `page` (optional): Page number (default: 1)
  - `limit` (optional): Items per page (default: 10)
- **Response:**
```json
{
  "data": [
    {
      "_id": "card-id",
      "productTitle": "Business Card",
      "price": 29.99,
      "category": {
        "_id": "category-id",
        "name": "Business"
      }
    }
  ],
  "limit": 10,
  "page": 1,
  "total": 50
}
```

---

### Order Routes

#### 1. Create Checkout Session
- **URL:** `POST /order/create-checkout-session`
- **Description:** Create Stripe checkout session
- **Authentication:** Required (Any role)
- **Request Body:**
```json
{
  "cardId": "card-id",
  "quantity": 1
}
```
- **Response:**
```json
{
  "sessionId": "stripe-session-id",
  "url": "stripe-checkout-url"
}
```

#### 2. Create Order
- **URL:** `POST /order/create-order`
- **Description:** Create a new order
- **Authentication:** Required (Any role)
- **Request Body:**
```json
{
  "cardId": "card-id",
  "frontImage": "base64-image-data",
  "backImage": "base64-image-data",
  "selectedVariant": "variant-id"
}
```
- **Response:**
```json
{
  "_id": "order-id",
  "frontImage": "base64-image-data",
  "backImage": "base64-image-data",
  "selectedVariant": "variant-id",
  "product": "card-id",
  "quantity": 1,
  "userId": "user-id",
  "paymentStatus": "created"
}
```

#### 3. Update Address in Order
- **URL:** `PUT /order/update-address/:id`
- **Description:** Update shipping address for an order
- **Authentication:** Required (Any role)
- **Request Body:**
```json
{
  "address": "123 Main St, City, State 12345",
  "orderId": "order-id"
}
```
- **Response:**
```json
{
  "_id": "order-id",
  "address": "123 Main St, City, State 12345",
  "orderId": "order-id"
}
```

#### 4. Post Payment URL
- **URL:** `PUT /order/post-payment-url/:id`
- **Description:** Update order status after payment
- **Authentication:** Required (Any role)
- **Response:**
```json
{
  "_id": "order-id",
  "orderStatus": "ordered",
  "paymentStatus": "done",
  "product": {
    "_id": "card-id",
    "productTitle": "Business Card"
  }
}
```

#### 5. Update Order
- **URL:** `PUT /order/update-order/:id`
- **Description:** Update order details
- **Authentication:** Required (Any role)
- **Request Body:**
```json
{
  "orderId": "new-order-id",
  "cardId": "new-card-id",
  "frontImage": "new-base64-image",
  "backImage": "new-base64-image",
  "selectedVariant": "new-variant-id"
}
```
- **Response:**
```json
{
  "message": "Order updated successfully"
}
```

#### 6. Get All Orders for Admin Panel
- **URL:** `GET /order/get-all-orders`
- **Description:** Get all orders for admin panel
- **Authentication:** Required
- **Query Parameters:**
  - `page` (optional): Page number (default: 1)
  - `limit` (optional): Items per page (default: 10)
- **Response:**
```json
{
  "data": [
    {
      "_id": "order-id",
      "userId": {
        "_id": "user-id",
        "name": "John Doe",
        "email": "john@example.com"
      },
      "product": {
        "_id": "card-id",
        "productTitle": "Business Card",
        "category": {
          "_id": "category-id",
          "name": "Business"
        }
      },
      "paymentStatus": "done",
      "orderStatus": "ordered"
    }
  ],
  "limit": 10,
  "page": 1,
  "total": 25
}
```

---

### Profile Routes

#### 1. Get Profile by Order ID
- **URL:** `GET /profile/get-profile/:orderId`
- **Description:** Get profile information by order ID
- **Authentication:** Required (Any role)
- **Response:**
```json
{
  "_id": "profile-id",
  "uniqueId": "123456",
  "name": "John Doe",
  "designation": "Developer",
  "companyName": "Tech Corp",
  "logo": "logo-filename.jpg",
  "orderId": {
    "_id": "order-id",
    "product": {
      "_id": "card-id",
      "productTitle": "Business Card"
    }
  }
}
```

#### 2. Update Profile
- **URL:** `PUT /profile/update-profile/:profileId`
- **Description:** Update profile information
- **Authentication:** Required (Any role)
- **Content-Type:** `multipart/form-data`
- **Request Body:**
```form-data
name: "John Doe"
designation: "Senior Developer"
companyName: "Tech Corp"
socialLink1: "messenger-link"
socialLink2: "twitter-link"
socialLink3: "instagram-link"
socialLink4: "linkedin-link"
socialLink5: "facebook-link"
url1: "https://website.com"
urlName1: "Website"
url2: "https://portfolio.com"
urlName2: "Portfolio"
url3: "https://blog.com"
urlName3: "Blog"
email: "john@example.com"
phone: "+1234567890"
username: "johndoe"
logo: [image file]
```
- **Response:**
```json
{
  "_id": "profile-id",
  "uniqueId": "johndoe",
  "name": "John Doe",
  "designation": "Senior Developer",
  "companyName": "Tech Corp"
}
```

#### 3. Get User Profile by Username (Public)
- **URL:** `GET /profile/profile/get-user/username/:username`
- **Description:** Get public profile by username
- **Authentication:** Not required
- **Response:**
```json
{
  "_id": "profile-id",
  "uniqueId": "johndoe",
  "name": "John Doe",
  "designation": "Developer",
  "companyName": "Tech Corp",
  "logo": "logo-filename.jpg",
  "orderId": {
    "_id": "order-id"
  }
}
```

#### 4. Create Profile
- **URL:** `POST /profile/create-profile`
- **Description:** Create a new profile
- **Authentication:** Required (Any role)
- **Content-Type:** `multipart/form-data`
- **Request Body:**
```form-data
name: "John Doe"
designation: "Developer"
cardId: "card-id"
companyName: "Tech Corp"
socialLink1: "messenger-link"
socialLink2: "twitter-link"
socialLink3: "instagram-link"
socialLink4: "linkedin-link"
socialLink5: "facebook-link"
url1: "https://website.com"
urlName1: "Website"
url2: "https://portfolio.com"
urlName2: "Portfolio"
url3: "https://blog.com"
urlName3: "Blog"
email: "john@example.com"
phone: "+1234567890"
fields: "field1,field2"
file: [image file]
```
- **Response:**
```json
{
  "_id": "profile-id",
  "uniqueId": "123456",
  "name": "John Doe",
  "designation": "Developer",
  "cardId": "card-id",
  "userId": "user-id"
}
```

#### 5. Delete Profile
- **URL:** `DELETE /profile/delete-profile/:profileId`
- **Description:** Delete a profile
- **Authentication:** Required (Any role)
- **Response:**
```json
{
  "message": "Profile deleted successfully"
}
```

#### 6. Get User Personal Profile
- **URL:** `GET /profile/get-user-personal-profile`
- **Description:** Get authenticated user's personal profile
- **Authentication:** Required (Any role)
- **Response:**
```json
{
  "_id": "user-id",
  "name": "John Doe",
  "email": "john@example.com",
  "phone": "+1234567890",
  "role": "user",
  "isVerified": true
}
```

#### 7. Save User Personal Profile
- **URL:** `PUT /profile/update`
- **Description:** Update user's personal information
- **Authentication:** Required (Any role)
- **Request Body:**
```json
{
  "name": "John Doe",
  "phone": "+1234567890",
  "addressLine1": "123 Main St",
  "addressLine2": "Apt 4B",
  "city": "New York",
  "state": "NY",
  "zipCode": "10001",
  "country": "USA"
}
```
- **Response:**
```json
{
  "_id": "user-id",
  "name": "John Doe",
  "email": "john@example.com",
  "phone": "+1234567890",
  "addressLine1": "123 Main St",
  "addressLine2": "Apt 4B",
  "city": "New York",
  "state": "NY",
  "zipCode": "10001",
  "country": "USA",
  "role": "user",
  "isVerified": true
}
```

#### 8. Check Username Availability
- **URL:** `POST /profile/check-username`
- **Description:** Check if username is available and save it
- **Authentication:** Required (Any role)
- **Request Body:**
```json
{
  "username": "johndoe",
  "profileId": "profile-id"
}
```
- **Response:**
```json
{
  "message": "Username saved successfully"
}
```

---

### Category Routes

#### 1. Create Category
- **URL:** `POST /category/category`
- **Description:** Create a new category (Admin only)
- **Authentication:** Required (Admin)
- **Content-Type:** `multipart/form-data`
- **Request Body:**
```form-data
name: "Business Cards"
file: [image file]
```
- **Response:**
```json
{
  "msg": "Category Created",
  "success": true
}
```

#### 2. Get All Categories
- **URL:** `GET /category/category`
- **Description:** Get all categories
- **Authentication:** Not required
- **Response:**
```json
{
  "msg": "Successfully",
  "success": true,
  "category": [
    {
      "_id": "category-id",
      "name": "Business Cards",
      "thumbnail": "thumbnail-path.jpg"
    }
  ]
}
```

#### 3. Update Category
- **URL:** `PUT /category/category/:id`
- **Description:** Update category information (Admin only)
- **Authentication:** Required (Admin)
- **Content-Type:** `multipart/form-data`
- **Request Body:**
```form-data
name: "Updated Business Cards"
file: [image file]
```
- **Response:**
```json
{
  "msg": "Successfully Updated",
  "success": true,
  "category": {
    "_id": "category-id",
    "name": "Updated Business Cards",
    "thumbnail": "new-thumbnail-path.jpg"
  }
}
```

#### 4. Delete Category
- **URL:** `DELETE /category/category/:id`
- **Description:** Delete a category (Admin only)
- **Authentication:** Required (Admin)
- **Response:**
```json
{
  "msg": "Successfully deleted",
  "success": true
}
```

---

### Tag Routes

#### 1. Create Tag
- **URL:** `POST /tags/tags`
- **Description:** Create a new tag (Admin only)
- **Authentication:** Required (Admin)
- **Content-Type:** `multipart/form-data`
- **Request Body:**
```form-data
name: "Professional"
file: [image file]
```
- **Response:**
```json
{
  "msg": "Tag Created",
  "success": true
}
```

#### 2. Get All Tags
- **URL:** `GET /tags/tags`
- **Description:** Get all tags
- **Authentication:** Not required
- **Response:**
```json
{
  "msg": "Successfully",
  "success": true,
  "tag": [
    {
      "_id": "tag-id",
      "name": "Professional",
      "thumbnail": "thumbnail-path.jpg"
    }
  ]
}
```

#### 3. Update Tag
- **URL:** `PUT /tags/tags/:id`
- **Description:** Update tag information (Admin only)
- **Authentication:** Required (Admin)
- **Content-Type:** `multipart/form-data`
- **Request Body:**
```form-data
name: "Updated Professional"
file: [image file]
```
- **Response:**
```json
{
  "msg": "Successfully Updated",
  "success": true,
  "tag": {
    "_id": "tag-id",
    "name": "Updated Professional",
    "thumbnail": "new-thumbnail-path.jpg"
  }
}
```

#### 4. Delete Tag
- **URL:** `DELETE /tags/tags/:id`
- **Description:** Delete a tag (Admin only)
- **Authentication:** Required (Admin)
- **Response:**
```json
{
  "msg": "Successfully deleted",
  "success": true
}
```

---

### Artist Routes

#### 1. Get All Artists
- **URL:** `GET /artist`
- **Description:** Get all artists
- **Authentication:** Required
- **Response:**
```json
[
  {
    "_id": "artist-id",
    "name": "Artist Name",
    "bio": "Artist biography",
    "image": "artist-image.jpg"
  }
]
```

#### 2. Get Artist by ID
- **URL:** `GET /artist/:id`
- **Description:** Get artist by ID
- **Authentication:** Required
- **Response:**
```json
{
  "_id": "artist-id",
  "name": "Artist Name",
  "bio": "Artist biography",
  "image": "artist-image.jpg"
}
```

#### 3. Create Artist
- **URL:** `POST /artist`
- **Description:** Create a new artist (Admin only)
- **Authentication:** Required (Admin)
- **Content-Type:** `multipart/form-data`
- **Request Body:**
```form-data
name: "Artist Name"
bio: "Artist biography"
file: [image file]
```
- **Response:**
```json
{
  "_id": "artist-id",
  "name": "Artist Name",
  "bio": "Artist biography",
  "image": "artist-image.jpg"
}
```

#### 4. Update Artist
- **URL:** `PUT /artist/:id`
- **Description:** Update artist information (Admin only)
- **Authentication:** Required (Admin)
- **Content-Type:** `multipart/form-data`
- **Request Body:**
```form-data
name: "Updated Artist Name"
bio: "Updated biography"
file: [image file]
```
- **Response:**
```json
{
  "_id": "artist-id",
  "name": "Updated Artist Name",
  "bio": "Updated biography",
  "image": "updated-image.jpg"
}
```

#### 5. Delete Artist
- **URL:** `DELETE /artist/:id`
- **Description:** Delete an artist (Admin only)
- **Authentication:** Required (Admin)
- **Response:**
```json
{
  "_id": "artist-id",
  "name": "Artist Name"
}
```

---

### Settings Routes

#### 1. Save Settings
- **URL:** `POST /setting/save-setting`
- **Description:** Save application settings (Admin only)
- **Authentication:** Required (Admin)
- **Request Body:**
```json
{
  "shippingCharges": 5.99
}
```
- **Response:**
```json
{
  "msg": "Setting Saved",
  "success": true
}
```

#### 2. Get Settings
- **URL:** `GET /setting`
- **Description:** Get application settings
- **Authentication:** Not required
- **Response:**
```json
{
  "msg": "Successfully",
  "success": true,
  "setting": {
    "_id": "setting-id",
    "shippingCharges": 5.99
  }
}
```

---

## Error Responses

### Common Error Format
```json
{
  "message": "Error description",
  "status": 400
}
```

### Common Error Codes
- `400` - Bad Request
- `401` - Unauthorized
- `403` - Forbidden
- `404` - Not Found
- `500` - Internal Server Error

### Common Error Messages
- `"Token required"` - Missing authentication token
- `"Invalid token"` - Invalid or expired token
- `"User not found"` - User doesn't exist
- `"Card not found"` - Card doesn't exist
- `"Order not found"` - Order doesn't exist
- `"Profile not found"` - Profile doesn't exist
- `"Invalid OTP"` - Wrong OTP provided
- `"OTP expired"` - OTP has expired

---

## Pagination

Most list endpoints support pagination with the following query parameters:
- `page` - Page number (default: 1)
- `limit` - Items per page (default: 10)

### Paginated Response Format
```json
{
  "data": [...],
  "limit": 10,
  "page": 1,
  "total": 100
}
```

---

## File Upload

For endpoints that accept file uploads:
- Use `multipart/form-data` content type
- File field name is typically `file`
- Supported formats: JPG, PNG, GIF, SVG
- Maximum file size: Check server configuration

---

## Rate Limiting

API endpoints may have rate limiting applied. Check with the server administrator for specific limits.

---

## Versioning

This documentation covers the current version of the API. For version-specific endpoints, the version number will be included in the URL path.

---

## Support

For API support and questions, please contact the development team or refer to the project documentation. 
