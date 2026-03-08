# Inventory Management System — API Documentation

**Base URL:** `http://localhost:8080`  
**Content-Type:** `application/json`  
**Auth:** Protected routes require `Authorization: Bearer <token>` header (JWT received from login)

---

## 🔐 Authentication

### 1. Register
```
POST /auth/register
```
**Request Body:**
```json
{
  "username": "john",
  "email": "john@example.com",
  "password": "Secret123!",
  "role": "staff"
}
```
> `role` options: `"admin"` | `"manager"` | `"staff"` (default: `"staff"`)  
> `warehouse_id` (integer, optional) — assign user to a specific warehouse

**Success Response `201`:**
```json
{ "message": "user registered successfully" }
```
**Error Responses:**
| Status | Meaning |
|--------|---------|
| `400` | Missing required fields |
| `409` | Email already registered |

---

### 2. Login
```
POST /auth/login
```
**Request Body:**
```json
{
  "email": "john@example.com",
  "password": "Secret123!"
}
```
**Success Response `200`:**
```json
{ "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." }
```
> ⚠️ **Store this token — pass it as `Authorization: Bearer <token>` in every protected request.**

**Error Responses:**
| Status | Meaning |
|--------|---------|
| `401` | Invalid email or password |

---

## 📦 Products

### 3. Create Product
```
POST /products
```
**Request Body:**
```json
{
  "name": "Widget A",
  "sku": "WGT-001",
  "price": 50.00,
  "description": "A sample product",
  "stock": 0,
  "reorder_level": 10
}
```
**Success Response `201`:**
```json
{
  "id": 1,
  "name": "Widget A",
  "sku": "WGT-001",
  "price": 50.00,
  "description": "A sample product",
  "stock": 0,
  "reorder_level": 10
}
```

---

### 4. List All Products
```
GET /products
```
**Success Response `200`:**
```json
[
  { "id": 1, "name": "Widget A", "sku": "WGT-001", "price": 50.00, "stock": 25, "reorder_level": 10 }
]
```

---

### 5. Get Product by ID
```
GET /products/get?id=1
```
**Success Response `200`:**
```json
{ "id": 1, "name": "Widget A", "sku": "WGT-001", "price": 50.00, "stock": 25, "reorder_level": 10 }
```

---

### 6. Update Product
```
PUT /products/update?id=1
```
**Request Body:** (same fields as create)
```json
{
  "name": "Widget A Updated",
  "sku": "WGT-001",
  "price": 55.00,
  "description": "Updated description",
  "stock": 25,
  "reorder_level": 15
}
```
**Success Response `200`:**
```json
{ "message": "product updated" }
```

---

### 7. Delete Product
```
DELETE /products/delete?id=1
```
**Success Response `200`:**
```json
{ "message": "product deleted" }
```

---

### 8. Get Low Stock Products
```
GET /products/low-stock
```
> Returns products where `stock <= reorder_level`

**Success Response `200`:**
```json
[
  { "id": 3, "name": "Widget C", "sku": "WGT-003", "stock": 2, "reorder_level": 10 }
]
```

---

## 🏭 Warehouses

### 9. Create Warehouse
```
POST /warehouses
```
**Request Body:**
```json
{
  "name": "Main Warehouse",
  "location": "Dhaka",
  "description": "Primary storage facility"
}
```
**Success Response `201`:**
```json
{
  "id": 1,
  "name": "Main Warehouse",
  "location": "Dhaka",
  "description": "Primary storage facility",
  "created_at": "2026-03-08T10:00:00Z",
  "updated_at": "2026-03-08T10:00:00Z"
}
```

---

### 10. List All Warehouses
```
GET /warehouses
```
**Success Response `200`:**
```json
[
  { "id": 1, "name": "Main Warehouse", "location": "Dhaka", "description": "...", "created_at": "...", "updated_at": "..." }
]
```

---

### 11. Update Warehouse
```
PUT /warehouses
```
**Request Body:**
```json
{
  "id": 1,
  "name": "Main Warehouse Updated",
  "location": "Chittagong",
  "description": "Updated description"
}
```
**Success Response `200`:** Updated warehouse object

---

### 12. Delete Warehouse
```
DELETE /warehouses?id=1
```
**Success Response `204`:** (no body)

---

## 🛒 Purchase Management — Stock IN  
> ⚠️ **All purchase routes require `Authorization: Bearer <token>` header**

### 13. Create Purchase (Stock IN) ⭐
```
POST /purchases
Authorization: Bearer <token>
```
**Request Body:**
```json
{
  "warehouse_id": 1,
  "items": [
    {
      "product_id": 1,
      "quantity": 50,
      "unit_price": 45.00
    },
    {
      "product_id": 2,
      "quantity": 20,
      "unit_price": 30.00
    }
  ]
}
```
> - `warehouse_id` — **required**, must exist in DB  
> - `items` — **required**, at least 1 item  
> - `quantity` — **must be > 0**  
> - `unit_price` — optional (can be omitted or null)  
> - Stock is **automatically incremented** for each product  
> - An **inventory movement log** is automatically created  
> - Entire operation is **transaction-safe** (all-or-nothing)

**Success Response `201`:**
```json
{
  "message": "purchase created successfully",
  "purchase": {
    "id": 5,
    "warehouse_id": 1,
    "created_by": 3,
    "created_at": "2026-03-08T17:00:00Z",
    "updated_at": "2026-03-08T17:00:00Z",
    "items": [
      { "id": 8, "purchase_id": 5, "product_id": 1, "quantity": 50, "unit_price": 45.00 },
      { "id": 9, "purchase_id": 5, "product_id": 2, "quantity": 20, "unit_price": 30.00 }
    ]
  }
}
```

**Error Responses:**
| Status | Body | Reason |
|--------|------|--------|
| `400` | `{"error":"warehouse_id is required"}` | Missing warehouse_id |
| `400` | `{"error":"items are required"}` | Empty items array |
| `400` | `{"error":"quantity must be greater than zero"}` | quantity ≤ 0 |
| `401` | `{"error":"authorization header required"}` | No/invalid JWT |
| `404` | `{"error":"warehouse not found"}` | warehouse_id doesn't exist |
| `404` | `{"error":"product not found: product_id=99"}` | product_id doesn't exist |
| `500` | `{"error":"internal server error"}` | DB error |

---

### 14. List All Purchases (Purchase History) ⭐
```
GET /purchases
Authorization: Bearer <token>
```
**Success Response `200`:**
```json
{
  "purchases": [
    {
      "id": 5,
      "warehouse_id": 1,
      "created_by": 3,
      "created_at": "2026-03-08T17:00:00Z",
      "updated_at": "2026-03-08T17:00:00Z",
      "items": [
        { "id": 8, "purchase_id": 5, "product_id": 1, "quantity": 50, "unit_price": 45.00 }
      ]
    }
  ],
  "total": 1
}
```

---

### 15. Get Single Purchase by ID ⭐
```
GET /purchases/get?id=5
Authorization: Bearer <token>
```
**Success Response `200`:**
```json
{
  "id": 5,
  "warehouse_id": 1,
  "created_by": 3,
  "created_at": "2026-03-08T17:00:00Z",
  "updated_at": "2026-03-08T17:00:00Z",
  "items": [
    { "id": 8, "purchase_id": 5, "product_id": 1, "quantity": 50, "unit_price": 45.00 }
  ]
}
```

**Error Responses:**
| Status | Meaning |
|--------|---------|
| `400` | Missing or invalid id |
| `404` | Purchase not found |

---

## 📊 Inventory Movements (Stock Change Log)
> ⚠️ **Requires `Authorization: Bearer <token>` header**

### 16. List Inventory Movements ⭐
```
GET /inventory-movements
Authorization: Bearer <token>
```
**Optional query params:**
- `?product_id=1` → filter by product
- `?warehouse_id=1` → filter by warehouse
- No params → returns all movements

**Success Response `200`:**
```json
{
  "movements": [
    {
      "id": 12,
      "product_id": 1,
      "warehouse_id": 1,
      "movement_type": "purchase",
      "quantity": 50,
      "reference_type": "purchase",
      "reference_id": 5,
      "created_by": 3,
      "notes": "",
      "created_at": "2026-03-08T17:00:00Z"
    }
  ],
  "total": 1
}
```
> `movement_type` values: `"purchase"` (stock in) | `"sale"` (stock out) | `"adjustment"` | `"transfer"`

---

## 📤 Stock OUT

### 17. Stock Out
```
POST /api/stock-out
```
> Used for recording stock going out (sales, consumption etc.)  
> Contact backend team for request body format.

---

## 🤖 ML / AI Agent

| Endpoint | Method | Description |
|----------|--------|-------------|
| `GET /ml/health` | GET | Check ML service health |
| `POST /ml/agent` | POST | Chat with AI inventory agent |
| `POST /ml/demand-forecast` | POST | Get demand forecast |
| `POST /ml/smart-reorder` | POST | Get smart reorder suggestions |
| `POST /ml/pricelist-optimize` | POST | Get price optimization |
| `POST /ml/full-analysis` | POST | Full inventory analysis |

---

## 🔧 Common Error Format
All errors return JSON in this format:
```json
{ "error": "error message here" }
```

## 📝 How to Use Auth Token (Frontend Example)
```javascript
// After login, store the token
const token = loginResponse.token;

// Use it in all protected requests
fetch('http://localhost:8080/purchases', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': `Bearer ${token}`
  },
  body: JSON.stringify({
    warehouse_id: 1,
    items: [
      { product_id: 1, quantity: 50, unit_price: 45.00 }
    ]
  })
});
```

---

## ⭐ Purchase Management (Stock IN) — Frontend Flow

```
1. GET /warehouses          → populate warehouse dropdown
2. GET /products            → populate product dropdown  
3. POST /auth/login         → get JWT token
4. POST /purchases          → submit purchase form (token required)
5. GET /purchases           → show purchase history page (token required)
6. GET /purchases/get?id=X  → show single purchase detail (token required)
7. GET /inventory-movements?product_id=X → show movement log (token required)
```
