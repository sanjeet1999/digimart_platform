## 📌 Buyer User Stories – DigiMart Platform

### Account & Authentication
- Buyer can sign up using email and password (no OTP required).
- Buyer can log in securely using verified credentials.
- Buyer can edit and update personal information in their profile.

### Product Browsing & Search
- Buyer can view a list of digital products with search and filter options.
- Buyer can navigate through products using pagination controls (next, previous, page numbers).
- Buyer can choose how many products to display per page (10, 20, 50 items).
- Buyer can sort products by price, date, rating, or popularity.
- Buyer can search for products by name, description, or category.

### Cart Management
- Buyer has no cart initially - cart is created only when first adding an item.
- Buyer can add digital products to their shopping cart for later purchase.
- Buyer can remove products from their cart individually or clear the entire cart.
- Buyer's cart shell is preserved even when emptied, maintaining shopping state.
- Buyer can continue shopping seamlessly after emptying cart without losing cart state.
- Buyer can view their cart contents with product details, prices, and total cost.
- Buyer can update product quantities in their cart (if applicable for bundled products).
- Buyer can view cart summary including subtotal, taxes, and final total.
- Buyer can buy all products in their cart in a single transaction.
- Buyer can selectively purchase specific products from their cart while keeping others.
- Buyer can save their cart for later and return to complete the purchase.
- Buyer's cart persists across sessions until explicitly deleted.

### Cart Lifecycle States
- **NO CART**: Buyer starts with no cart - clean and efficient.
- **CART CREATED**: Cart is created only when buyer first adds an item to cart.
- **CART EMPTIED**: When buyer clears cart, items are removed but cart shell is preserved.
- **CART REFILLED**: Buyer can add new items to existing cart shell without recreation.
- **CART PERSISTENT**: Cart remains available across browser sessions and app restarts.

### Address Management & Purchase for Others
- Buyer can manage multiple email addresses in their address management system.
- Buyer can add, edit, and delete email addresses for purchase recipients.
- Buyer can purchase products for themselves or for someone else using saved email addresses.
- Buyer can select recipient email address during checkout process.
- Buyer must verify recipient email address with OTP before completing purchase for others and for himself as well since we have not verified his email as well during signup.

### Purchase & Payment
- Buyer must verify their email with OTP before making any purchase.
- Buyer can make payments and instantly access digital products after OTP verification.
- Buyer can download purchased products immediately after payment.
- Buyer receives email confirmation and access link after each purchase.

### Reviews & Feedback
- Buyer can rate and review products after logging in.
- Buyer can give star ratings and upload review images (stored in S3 bucket).

### Order Management
- Buyer can view their entire order history with download access.
- Buyer can view purchase history for products bought for themselves and others.



## 🛍️ Seller User Stories – Digital Goods E-commerce Platform

- Seller can sign up using email and password (no OTP required).
- Seller can log in securely using verified credentials.
- Seller can create and manage their profile with bio, avatar, and contact details.
- Seller can upload new digital products (eBooks, PDFs, art, videos).
- Seller can set title, description, price, categories, and tags for each product.
- Seller can edit or delete their existing products.
- Seller can view all their uploaded products in a dashboard with pagination.
- Seller can navigate through their products using page controls when they have many items.
- Seller can sort their products by date, price, or sales performance.
- Seller can search through their own products by name or category.
- Seller can view sales history with order IDs and buyer info.
- Seller can track total earnings and download sales reports.
- Seller receives notification/email upon a successful product sale.
- Seller can respond to buyer reviews or feedback on their products.
- Seller can mark products as free, discounted, or on promotion.
- Seller can see analytics for each product (downloads, ratings).
