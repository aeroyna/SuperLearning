# E-Commerce Database Design

## Requirements Analysis

```
┌─────────────────────────────────────────────────────────────────┐
│              E-Commerce Requirements                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  FUNCTIONAL REQUIREMENTS                                        │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ • User registration and authentication                    │ │
│  │ • Product catalog with search and filtering               │ │
│  │ • Shopping cart management                                 │ │
│  │ • Order processing and tracking                           │ │
│  │ • Inventory management                                     │ │
│  │ • Payment processing                                       │ │
│  │ • Reviews and ratings                                      │ │
│  │ • Recommendations                                          │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  NON-FUNCTIONAL REQUIREMENTS                                    │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ • High availability (99.9%+)                               │ │
│  │ • Sub-second page loads                                    │ │
│  │ • Handle traffic spikes (Black Friday)                     │ │
│  │ • Strong consistency for orders/payments                   │ │
│  │ • Eventually consistent for catalog/reviews               │ │
│  │ • ACID transactions for inventory                          │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  ACCESS PATTERNS                                                │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Read-heavy: Product browsing, search (80% reads)          │ │
│  │ Write-heavy: Cart updates, order placement (20% writes)    │ │
│  │ Hot data: Featured products, active carts                  │ │
│  │ Cold data: Old orders, archived products                   │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Core Data Model

```
┌─────────────────────────────────────────────────────────────────┐
│              E-Commerce Entity Relationships                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────┐     ┌──────────────┐     ┌──────────────┐        │
│  │  Users   │────<│   Orders     │>────│ Order_Items  │        │
│  └──────────┘     └──────────────┘     └──────────────┘        │
│       │                 │                      │                │
│       │                 │                      │                │
│       ▼                 ▼                      ▼                │
│  ┌──────────┐     ┌──────────────┐     ┌──────────────┐        │
│  │Addresses │     │  Payments    │     │  Products    │        │
│  └──────────┘     └──────────────┘     └──────────────┘        │
│                                              │                  │
│                         ┌────────────────────┤                  │
│                         │                    │                  │
│                         ▼                    ▼                  │
│                   ┌──────────┐        ┌──────────────┐         │
│                   │Categories│        │  Inventory   │         │
│                   └──────────┘        └──────────────┘         │
│                                                                  │
│  Additional entities: Reviews, Cart, Wishlist, Coupons          │
└─────────────────────────────────────────────────────────────────┘
```

## Schema Design (PostgreSQL)

```sql
-- Users and Authentication
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    first_name VARCHAR(100),
    last_name VARCHAR(100),
    phone VARCHAR(20),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_users_email ON users(email);

-- Addresses
CREATE TABLE addresses (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id),
    type VARCHAR(20) DEFAULT 'shipping',
    street_address TEXT NOT NULL,
    city VARCHAR(100) NOT NULL,
    state VARCHAR(100),
    postal_code VARCHAR(20) NOT NULL,
    country VARCHAR(100) NOT NULL,
    is_default BOOLEAN DEFAULT false
);

CREATE INDEX idx_addresses_user ON addresses(user_id);

-- Products
CREATE TABLE products (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    sku VARCHAR(50) UNIQUE NOT NULL,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    price DECIMAL(10,2) NOT NULL,
    compare_at_price DECIMAL(10,2),
    category_id UUID REFERENCES categories(id),
    brand VARCHAR(100),
    attributes JSONB DEFAULT '{}',
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_products_sku ON products(sku);
CREATE INDEX idx_products_category ON products(category_id);
CREATE INDEX idx_products_price ON products(price);
CREATE INDEX idx_products_attributes ON products USING GIN(attributes);

-- Inventory (per warehouse/location)
CREATE TABLE inventory (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    product_id UUID REFERENCES products(id),
    warehouse_id UUID REFERENCES warehouses(id),
    quantity INTEGER NOT NULL DEFAULT 0,
    reserved INTEGER NOT NULL DEFAULT 0,
    CONSTRAINT positive_quantity CHECK (quantity >= 0),
    CONSTRAINT positive_reserved CHECK (reserved >= 0),
    UNIQUE(product_id, warehouse_id)
);

-- Orders
CREATE TABLE orders (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id),
    order_number VARCHAR(50) UNIQUE NOT NULL,
    status VARCHAR(50) DEFAULT 'pending',
    subtotal DECIMAL(12,2) NOT NULL,
    tax DECIMAL(12,2) DEFAULT 0,
    shipping_cost DECIMAL(12,2) DEFAULT 0,
    total DECIMAL(12,2) NOT NULL,
    shipping_address_id UUID REFERENCES addresses(id),
    billing_address_id UUID REFERENCES addresses(id),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_orders_user ON orders(user_id);
CREATE INDEX idx_orders_status ON orders(status);
CREATE INDEX idx_orders_created ON orders(created_at);

-- Order Items
CREATE TABLE order_items (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    order_id UUID REFERENCES orders(id),
    product_id UUID REFERENCES products(id),
    quantity INTEGER NOT NULL,
    unit_price DECIMAL(10,2) NOT NULL,
    total_price DECIMAL(12,2) NOT NULL
);

CREATE INDEX idx_order_items_order ON order_items(order_id);
```

## Technology Stack

```
┌─────────────────────────────────────────────────────────────────┐
│              Recommended Technology Stack                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                     Application                          │    │
│  └─────────────────────────┬───────────────────────────────┘    │
│                            │                                     │
│  ┌─────────────────────────┼───────────────────────────────┐    │
│  │                         │                               │    │
│  │    ┌────────────────────┼────────────────────┐         │    │
│  │    │                    │                    │          │    │
│  │    ▼                    ▼                    ▼          │    │
│  │                                                          │    │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │    │
│  │  │  PostgreSQL  │  │    Redis     │  │Elasticsearch │   │    │
│  │  │              │  │              │  │              │   │    │
│  │  │ • Users      │  │ • Sessions   │  │ • Product    │   │    │
│  │  │ • Orders     │  │ • Cart       │  │   Search     │   │    │
│  │  │ • Inventory  │  │ • Rate Limit │  │ • Facets     │   │    │
│  │  │ • Payments   │  │ • Cache      │  │ • Autocomplete│  │    │
│  │  │              │  │ • Pub/Sub    │  │              │   │    │
│  │  └──────────────┘  └──────────────┘  └──────────────┘   │    │
│  │                                                          │    │
│  │  Primary (ACID)    Performance       Search              │    │
│  │                                                          │    │
│  └──────────────────────────────────────────────────────────┘    │
│                                                                  │
│  Additional:                                                    │
│  • S3/CDN for product images                                    │
│  • Kafka for order events                                       │
│  • ClickHouse for analytics                                     │
└─────────────────────────────────────────────────────────────────┘
```

## Inventory Management Pattern

```
┌─────────────────────────────────────────────────────────────────┐
│              Inventory Reservation Pattern                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Problem: Prevent overselling during high traffic               │
│                                                                  │
│  Solution: Reserve → Confirm pattern with transactions          │
│                                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                                                             │ │
│  │  1. ADD TO CART (soft reserve)                             │ │
│  │     UPDATE inventory                                        │ │
│  │     SET reserved = reserved + quantity                     │ │
│  │     WHERE product_id = ? AND                               │ │
│  │           quantity - reserved >= requested_qty;            │ │
│  │                                                             │ │
│  │  2. CHECKOUT (hard commit)                                 │ │
│  │     BEGIN TRANSACTION;                                      │ │
│  │       UPDATE inventory                                      │ │
│  │       SET quantity = quantity - qty,                       │ │
│  │           reserved = reserved - qty                        │ │
│  │       WHERE product_id = ?;                                │ │
│  │                                                             │ │
│  │       INSERT INTO orders (...);                            │ │
│  │       INSERT INTO order_items (...);                       │ │
│  │     COMMIT;                                                 │ │
│  │                                                             │ │
│  │  3. CART EXPIRATION (release reserve)                      │ │
│  │     UPDATE inventory                                        │ │
│  │     SET reserved = reserved - qty                          │ │
│  │     WHERE product_id = ?;                                  │ │
│  │                                                             │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  Use Redis TTL for cart expiration triggers                     │
└─────────────────────────────────────────────────────────────────┘
```

## Scaling Strategies

```
┌─────────────────────────────────────────────────────────────────┐
│              Scaling for E-Commerce                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  READ SCALING                                                   │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ • PostgreSQL read replicas for product queries            │ │
│  │ • Redis caching for product details (TTL: 1 hour)         │ │
│  │ • CDN for product images                                   │ │
│  │ • Elasticsearch for search (scales horizontally)          │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  WRITE SCALING                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ • Partition orders by date (monthly partitions)           │ │
│  │ • Shard by user_id or region for multi-tenant             │ │
│  │ • Async processing for non-critical writes                │ │
│  │ • Queue orders during peak (Kafka)                        │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  CACHING STRATEGY                                               │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Layer 1: Browser/CDN cache (static assets)                │ │
│  │ Layer 2: Application cache (product catalog)              │ │
│  │ Layer 3: Redis (sessions, cart, hot products)             │ │
│  │ Layer 4: Query result cache (complex queries)             │ │
│  │                                                             │ │
│  │ Cache invalidation: Event-driven via Kafka/Redis Pub/Sub  │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```
