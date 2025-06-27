# EasyShop E-Commerce API

![ChatGPT Image Jun 27, 2025, 04_06_36 AM](https://github.com/user-attachments/assets/41ee6cdc-9a38-42e1-a6d6-c1fbe403d880)


## 📝 Project Description
EasyShop is a Spring Boot-based e-commerce API backend for an online store. This project serves as the backend system for Version 2 of the EasyShop platform, building upon an existing operational system (Version 1). The API handles user authentication, product management, shopping cart functionality, and order processing.
## 🚀 Key Features

- 🔒 User registration and authentication (JWT token-based)
- 📦 Product catalog with search/filter capabilities
- 🛒 Shopping cart functionality
- 💳 Order processing system
- 👔 Admin-only product and category management

## 🏗️📊  Core Functionality

- User registration and login with JWT tokens
- Product browsing with search/filter capabilities
- Shopping cart persistence across sessions
- Order creation from cart items
- Admin-only CRUD operations for products/categories

## 🐛🔧 Bugs Identified and Fixed

### 🐞 Bug 1: Faulty Product Search

**Problem:**  
Product search returned incorrect results when using filters.

**Symptoms:**
- Incorrect results when filtering by category
- Price range filters not working properly
- Color filters returning partial matches

**Root Cause:**
- SQL queries weren't properly combining multiple filter conditions
- Missing parameter validation in DAO layer

**Fix:**
```java
// Before (buggy implementation)
public List<Product> searchProducts(Integer categoryId, BigDecimal minPrice, 
                                  BigDecimal minPrice, String color) {
    String sql = "SELECT * FROM products WHERE 1=1";
    
    if(categoryId != null) {
        sql += " OR category_id = ?"; // Wrong operator (OR instead of AND)
    }
    // ... other conditions
}

// After (fixed implementation)
public List<Product> searchProducts(Integer categoryId, BigDecimal minPrice,
                                  BigDecimal maxPrice, String color) {
    String sql = "SELECT * FROM products WHERE 1=1";
    List<Object> params = new ArrayList<>();
    
    if(categoryId != null) {
        sql += " AND category_id = ?";
        params.add(categoryId);
    }
    if(minPrice != null) {
        sql += " AND price >= ?";
        params.add(minPrice);
    }
    if(maxPrice != null) {
        sql += " AND price <= ?";
        params.add(maxPrice);
    }
    if(color != null && !color.isEmpty()) {
        sql += " AND color = ?";
        params.add(color);
    }
    
    return jdbcTemplate.query(sql, new ProductRowMapper(), params.toArray());
}
```
## 🪲 Bug 2: Product Duplication During Updates 

**Problem:
Updating products created duplicates instead of modifying existing ones.**

**Solution:**

```java
@Transactional
public Product updateProduct(Product product) {
    String sql = """
        UPDATE products 
        SET name = ?, price = ?, description = ?,
            category_id = ?, color = ?, image_url = ?,
            stock = ?, featured = ?
        WHERE product_id = ?""";
    
    jdbcTemplate.update(sql,
        product.getName(),
        product.getPrice(),
        product.getDescription(),
        product.getCategoryId(),
        product.getColor(),
        product.getImageUrl(),
        product.getStock(),
        product.isFeatured(),
        product.getProductId());
    
    return getProductById(product.getProductId());
}
```
