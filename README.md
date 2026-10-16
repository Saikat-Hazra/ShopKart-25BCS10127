# 🛒 ShopKart — Full-Stack E-Commerce Application

**Author:** Saikat Hazra
**Repository:** ShopKart-25BCS10127
**Stack:** MERN (MongoDB, Express.js, React, Node.js)

> A complete, production-shaped e-commerce purchase journey — built incrementally across six engineering labs, covering authentication, product discovery, wishlist, cart, checkout and a real (Test Mode) payment integration with Razorpay.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Features by Lab](#features-by-lab)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Backend Setup](#1-backend-setup)
  - [Frontend Setup](#2-frontend-setup)
  - [Seeding Sample Products](#3-seeding-sample-products)
- [Environment Variables](#environment-variables)
- [API Reference](#api-reference)
- [Data Models](#data-models)
- [Authentication Flow](#authentication-flow)
- [Payment Flow (Razorpay Test Mode)](#payment-flow-razorpay-test-mode)
- [Key Engineering Decisions](#key-engineering-decisions)
- [Testing the App](#testing-the-app)
- [Known Limitations](#known-limitations)

---

## Overview

ShopKart is a fictional e-commerce startup's customer-facing shopping flow, built end-to-end:

```text
Register / Login
      ↓
Browse & Search Products
      ↓
Wishlist
      ↓
Shopping Cart
      ↓
Checkout (Shipping Details)
      ↓
Razorpay Payment (Test Mode)
      ↓
Order Confirmation
      ↓
Order History
```

Each stage was built as a standalone engineering lab and merged into this single repository. The goal of the project was not the shopping website itself, but demonstrating the underlying engineering concepts: authenticated REST APIs, MongoDB relationship modelling, global frontend state, server-side business-rule validation, and secure payment verification.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | React 18 (Vite), React Router v6, Axios |
| Backend | Node.js, Express.js |
| Database | MongoDB (MongoDB Atlas), Mongoose ODM |
| Authentication | JWT (jsonwebtoken) + HttpOnly cookies |
| Password Security | bcrypt |
| Payments | Razorpay (Test Mode) |
| State Management | React Context API |

**Constraint followed throughout:** no authentication frameworks (Passport.js, Clerk, Firebase Auth, Auth0) — auth is implemented from scratch with JWT + bcrypt + HttpOnly cookies, per the lab's technical requirements.

---

## Project Structure

```text
shopkart-full/
│
├── backend/
│   ├── config/
│   │   └── razorpay.js            # Razorpay client instance
│   ├── controllers/
│   │   ├── customer.controller.js # register, login, profile, logout, change password
│   │   ├── product.controller.js  # list/search/filter, get by id, create
│   │   ├── wishlist.controller.js # add, get, remove
│   │   ├── cart.controller.js     # add, get, update quantity, remove
│   │   └── order.controller.js    # create payment order, verify payment, list, get by id
│   ├── middlewares/
│   │   └── auth.middleware.js     # JWT verification, attaches req.user
│   ├── models/
│   │   ├── customer.model.js      # User doc — includes embedded wishlist + cart
│   │   ├── product.model.js
│   │   └── order.model.js         # purchase-time snapshot, not live references
│   ├── routes/
│   │   ├── customer.routes.js
│   │   ├── product.routes.js
│   │   ├── wishlist.routes.js
│   │   ├── cart.routes.js
│   │   └── order.routes.js
│   ├── seed/
│   │   └── seedProducts.js        # loads demo products into MongoDB
│   ├── utils/
│   │   └── generateToken.js
│   ├── index.js                   # app entry point
│   ├── .env.example
│   └── package.json
│
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   │   ├── Navbar.jsx
│   │   │   ├── ProductCard.jsx
│   │   │   ├── CartItem.jsx
│   │   │   └── ProtectedRoute.jsx
│   │   ├── context/
│   │   │   ├── AuthContext.jsx    # session state, login/register/logout
│   │   │   └── CartContext.jsx    # single shared cart state (Navbar, Cart page, Product cards)
│   │   ├── pages/
│   │   │   ├── Register.jsx
│   │   │   ├── Login.jsx
│   │   │   ├── Products.jsx
│   │   │   ├── Wishlist.jsx
│   │   │   ├── Cart.jsx
│   │   │   ├── Checkout.jsx       # shipping form + Razorpay Checkout integration
│   │   │   ├── OrderSuccess.jsx
│   │   │   ├── Orders.jsx
│   │   │   └── OrderDetails.jsx
│   │   ├── services/
│   │   │   └── api.js             # Axios instance, withCredentials: true
│   │   ├── App.jsx                # routes
│   │   └── main.jsx
│   ├── .env.example
│   └── package.json
│
└── README.md
```

MVC architecture is followed on the backend throughout: **Models** define schema and validation, **Controllers** hold business logic, **Routes** map HTTP verbs/paths to controllers, and **Middlewares** handle cross-cutting concerns (authentication).

---

## Features by Lab

| Lab | Feature | Status |
|---|---|:---:|
| 01 | Customer registration, login, JWT + HttpOnly cookie auth, protected profile route, logout | ✅ |
| 01 (Bonus) | Change password | ✅ |
| 03 | Product catalog, search, category/price filtering | ✅ |
| 04 | Wishlist (add, view, remove) | ✅ |
| 05 | Shopping cart (add, update quantity, remove), stock validation, global cart state, derived subtotal/item-count | ✅ |
| 06 | Checkout with shipping form validation, server-side order + stock + price revalidation, Razorpay Test Mode payment, signature verification, order history | ✅ |

---

## Getting Started

### Prerequisites

- [Node.js](https://nodejs.org/) v18 or later
- A [MongoDB Atlas](https://www.mongodb.com/cloud/atlas) account (free tier is sufficient) — or a local MongoDB instance
- A [Razorpay](https://dashboard.razorpay.com/signup) account (Test Mode keys only — no business verification needed)

### 1. Backend Setup

```bash
cd backend
npm install
```

Fill in `backend/.env` (see [Environment Variables](#environment-variables) below for what each value means), then:

```bash
npm run dev
```

Expected output:

```text
Connected to MongoDB
ShopKart server running on http://localhost:3000
```

### 2. Frontend Setup

In a **second terminal**:

```bash
cd frontend
npm install
npm run dev
```

The app opens at `http://localhost:5173`.

### 3. Seeding Sample Products

To populate the catalog with demo products (required before testing Cart/Checkout, since you need something to add):

```bash
cd backend
npm run seed
```

This clears and reloads a fixed set of 8 sample products into MongoDB.

---

## Environment Variables

### `backend/.env`

| Variable | Description |
|---|---|
| `PORT` | Port the Express server listens on (default `3000`) |
| `MONGO_URI` | MongoDB Atlas (or local) connection string. Non-SRV format recommended if `mongodb+srv://` fails on restrictive networks |
| `JWT_SECRET` | Any long random string used to sign/verify JWTs — generate with `node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"` |
| `JWT_EXPIRY` | Token lifetime, e.g. `1d` |
| `NODE_ENV` | `development` or `production` — controls the `secure` flag on cookies |
| `CLIENT_URL` | Frontend origin allowed by CORS (default `http://localhost:5173`) |
| `RAZORPAY_KEY_ID` | Test Mode Key ID from the Razorpay Dashboard (`rzp_test_...`) |
| `RAZORPAY_KEY_SECRET` | Test Mode Key Secret — **backend only, never exposed to the frontend** |

### `frontend/.env`

| Variable | Description |
|---|---|
| `VITE_API_URL` | Base URL of the backend API (default `http://localhost:3000`) |

> `.env` is excluded from version control via `.gitignore` in both `backend/` and `frontend/`. Only `.env.example` (with placeholder values) is committed.

---

## API Reference

### Customers (`/customers`)

| Method | Endpoint | Auth | Description |
|---|---|:---:|---|
| POST | `/customers/register` | ❌ | Create a new account |
| POST | `/customers/login` | ❌ | Authenticate, sets HttpOnly cookie |
| GET | `/customers/me` | ✅ | Get current user's profile |
| POST | `/customers/logout` | ✅ | Clears the auth cookie |
| PATCH | `/customers/change-password` | ✅ | Change password (bonus) |

### Products (`/products`)

| Method | Endpoint | Auth | Description |
|---|---|:---:|---|
| GET | `/products?search=&category=&minPrice=&maxPrice=` | ❌ | List/search/filter products |
| GET | `/products/:id` | ❌ | Get a single product |
| POST | `/products` | ❌ | Create a product (demo/seed use) |

### Wishlist (`/wishlist`)

| Method | Endpoint | Auth | Description |
|---|---|:---:|---|
| POST | `/wishlist/:productId` | ✅ | Add a product to wishlist |
| GET | `/wishlist` | ✅ | Get the current user's wishlist (populated) |
| DELETE | `/wishlist/:productId` | ✅ | Remove a product from wishlist |

### Cart (`/cart`)

| Method | Endpoint | Auth | Description |
|---|---|:---:|---|
| POST | `/cart/:productId` | ✅ | Add to cart (or increment if already present) |
| GET | `/cart` | ✅ | Get current cart (populated with product data) |
| PATCH | `/cart/:productId` | ✅ | Update quantity — body: `{ "quantity": number }` |
| DELETE | `/cart/:productId` | ✅ | Remove item from cart |

### Orders (`/orders`)

| Method | Endpoint | Auth | Description |
|---|---|:---:|---|
| POST | `/orders/create-payment-order` | ✅ | Validates cart/stock/prices, creates a pending order + Razorpay order |
| POST | `/orders/verify-payment` | ✅ | Verifies Razorpay signature, marks order PAID, clears cart |
| GET | `/orders` | ✅ | List the current user's orders |
| GET | `/orders/:id` | ✅ | Get a single order (ownership-checked) |

> All `✅` routes require a valid `token` cookie, attached automatically by the browser once logged in. Requests from the frontend use `axios` with `withCredentials: true` to carry this cookie.

---

## Data Models

### Customer

```js
{
  fullName: String,
  email: String,        // unique
  password: String,     // bcrypt hash, never returned in API responses
  phone: String,
  wishlist: [ObjectId],            // references to Product
  cart: [{ product: ObjectId, quantity: Number }],
  createdAt: Date
}
```

### Product

```js
{
  name: String,
  description: String,
  price: Number,
  image: String,
  category: String,
  stock: Number,
  createdAt, updatedAt
}
```

### Order

```js
{
  user: ObjectId,                  // references Customer
  items: [{
    product: ObjectId,
    name: String,                  // snapshot — not re-fetched from Product later
    price: Number,                 // snapshot at purchase time
    quantity: Number,
    image: String
  }],
  shippingAddress: { fullName, phone, addressLine1, city, state, pincode },
  totalAmount: Number,              // calculated server-side, never trusted from client
  paymentStatus: "PENDING" | "PAID" | "FAILED",
  status: "PENDING_PAYMENT" | "PLACED" | "CONFIRMED" | "SHIPPED" | "DELIVERED",
  razorpayOrderId: String,
  razorpayPaymentId: String,
  createdAt, updatedAt
}
```

**Why Order snapshots data instead of only storing references:** if a product's price changes after a purchase, the customer's order history must still reflect what they actually paid — not the product's current price. The Cart, by contrast, always shows live product data since nothing has been purchased yet.

---

## Authentication Flow

1. **Register** — password is hashed with `bcrypt` (10 salt rounds) before being stored; the plain password is never persisted.
2. **Login** — credentials are verified with `bcrypt.compare()`; on success, a JWT (`{ id: customer._id }` payload only) is signed and set as an **HttpOnly** cookie, so client-side JavaScript cannot read or exfiltrate it (XSS mitigation).
3. **Protected routes** — `auth.middleware.js` reads the `token` cookie, verifies it with `jsonwebtoken`, loads the corresponding customer, and attaches it to `req.user` for the controller to use.
4. **Logout** — clears the cookie server-side.

Login failures always return a generic `"Invalid email or password"` message — the API never reveals whether the email or the password was the incorrect part, to avoid helping an attacker enumerate valid accounts.

---

## Payment Flow (Razorpay Test Mode)

```text
Checkout (shipping form)
      ↓
POST /orders/create-payment-order
      ↓
Backend re-validates cart, stock, and prices from LIVE Product data
      ↓
Backend calculates totalAmount itself (frontend's number is ignored)
      ↓
Backend creates a ShopKart Order (status: PENDING_PAYMENT)
      ↓
Backend creates a matching Razorpay Order (amount in paise)
      ↓
Frontend opens Razorpay Checkout
      ↓
Test payment completed (card 4111 1111 1111 1111, any future expiry/CVV/OTP)
      ↓
Razorpay returns payment_id + order_id + signature to the frontend
      ↓
POST /orders/verify-payment
      ↓
Backend recomputes the HMAC-SHA256 signature using RAZORPAY_KEY_SECRET
and the Razorpay order id IT stored (not the one blindly sent by the client)
      ↓
Signature matches → order marked PAID / PLACED, cart cleared
Signature mismatch → order marked FAILED, cart left untouched
```

**Security rules enforced:**
- `RAZORPAY_KEY_SECRET` never leaves the backend (not sent in any API response, not referenced in frontend code)
- The cart is cleared **only** after a verified payment — never when Razorpay Checkout merely opens, and never on an unverified client-side success callback
- The server-calculated total is what's sent to Razorpay — a manipulated frontend `totalAmount` has no effect
- Signature verification always uses the Razorpay order ID stored in MongoDB at order-creation time, not whatever value the browser callback supplies

---

## Key Engineering Decisions

- **Global Cart state (Context API):** `CartContext.jsx` is the single source of truth for cart data across the Navbar, Product cards, and Cart page — avoiding the bug where different parts of the UI show different cart counts after a mutation.
- **Derived values, never stored:** cart subtotal and item count are computed from `cartItems` on every render (`Σ price × quantity`), not persisted as separate fields that could go stale.
- **Wishlist state hydration:** product cards fetch the user's actual saved wishlist on page load (rather than tracking only in-session clicks), so the "saved" heart icon correctly persists across refreshes.
- **MVC separation on the backend:** routes only wire up middleware and controllers; controllers hold all business logic; models hold schema/validation — makes each piece independently testable and easy to reason about in a viva.

---

## Testing the App

1. Register a new account → log in
2. Browse/search products on the home page
3. Click the heart icon to save a product to your Wishlist
4. Click "Add to Cart", adjust quantities from the Cart page
5. Click "Proceed to Checkout", fill in shipping details
6. Click "Place Order" → Razorpay Test Checkout opens
7. Use test card `4111 1111 1111 1111`, any future expiry date, any CVV, any OTP — **no real money is charged in Test Mode**
8. On success, you land on the Order Confirmation page; the cart is now empty and the order appears under "My Orders"

Backend endpoints can also be tested directly with Postman — see [API Reference](#api-reference) above for the full endpoint list and required bodies.

---

## Known Limitations

- Product creation (`POST /products`) is open and unauthenticated — there's no admin role system in this lab series, so it exists purely for seeding/demo purposes and should not be treated as production-ready.
- No pagination on the product listing or order history — acceptable at demo scale, would need addressing for a real catalog size.
- No image upload — product images are external URLs set during seeding.
- No email verification or password-reset flow — out of scope for this lab series.

---

<div align="center">

**Built as part of the ShopKart engineering lab series (Labs 01–06).**

</div>
