# Схема бази даних

## Загальний опис
Схема бази даних для маркетплейсу на Laravel з підтримкою багатомовності та різних ролей користувачів.

## Основні таблиці

### 1. Користувачі та ролі

#### users
```sql
CREATE TABLE users (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    email_verified_at TIMESTAMP NULL,
    password VARCHAR(255) NOT NULL,
    phone VARCHAR(20) NULL,
    avatar VARCHAR(255) NULL,
    status ENUM('active', 'inactive', 'banned') DEFAULT 'active',
    remember_token VARCHAR(100) NULL,
    created_at TIMESTAMP NULL,
    updated_at TIMESTAMP NULL
);
```

#### roles
```sql
CREATE TABLE roles (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    guard_name VARCHAR(255) NOT NULL,
    created_at TIMESTAMP NULL,
    updated_at TIMESTAMP NULL,
    UNIQUE KEY roles_name_guard_name_unique (name, guard_name)
);
```

#### model_has_roles
```sql
CREATE TABLE model_has_roles (
    role_id BIGINT UNSIGNED NOT NULL,
    model_type VARCHAR(255) NOT NULL,
    model_id BIGINT UNSIGNED NOT NULL,
    PRIMARY KEY (role_id, model_id, model_type),
    FOREIGN KEY (role_id) REFERENCES roles(id) ON DELETE CASCADE
);
```

### 2. Мови та локалізація

#### languages
```sql
CREATE TABLE languages (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    code VARCHAR(10) NOT NULL,
    locale VARCHAR(10) NOT NULL,
    flag VARCHAR(255) NULL,
    status BOOLEAN DEFAULT TRUE,
    sort_order INT DEFAULT 0,
    created_at TIMESTAMP NULL,
    updated_at TIMESTAMP NULL
);
```

### 3. Категорії

#### categories
```sql
CREATE TABLE categories (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    parent_id BIGINT UNSIGNED NULL,
    sort_order INT DEFAULT 0,
    status BOOLEAN DEFAULT TRUE,
    image VARCHAR(255) NULL,
    icon VARCHAR(255) NULL,
    created_at TIMESTAMP NULL,
    updated_at TIMESTAMP NULL,
    FOREIGN KEY (parent_id) REFERENCES categories(id) ON DELETE SET NULL
);
```

#### category_descriptions
```sql
CREATE TABLE category_descriptions (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    category_id BIGINT UNSIGNED NOT NULL,
    language_id BIGINT UNSIGNED NOT NULL,
    name VARCHAR(255) NOT NULL,
    slug VARCHAR(255) NOT NULL,
    description TEXT NULL,
    meta_title VARCHAR(255) NULL,
    meta_description TEXT NULL,
    meta_keywords VARCHAR(255) NULL,
    created_at TIMESTAMP NULL,
    updated_at TIMESTAMP NULL,
    FOREIGN KEY (category_id) REFERENCES categories(id) ON DELETE CASCADE,
    FOREIGN KEY (language_id) REFERENCES languages(id) ON DELETE CASCADE,
    UNIQUE KEY category_language_unique (category_id, language_id)
);
```

### 4. Продавці

#### sellers
```sql
CREATE TABLE sellers (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    user_id BIGINT UNSIGNED NOT NULL,
    company_name VARCHAR(255) NULL,
    tax_number VARCHAR(100) NULL,
    description TEXT NULL,
    logo VARCHAR(255) NULL,
    banner VARCHAR(255) NULL,
    address TEXT NULL,
    status ENUM('pending', 'active', 'inactive', 'banned') DEFAULT 'pending',
    commission_rate DECIMAL(5,2) DEFAULT 0.00,
    auto_approve_products BOOLEAN DEFAULT FALSE,
    total_sales DECIMAL(15,2) DEFAULT 0.00,
    total_orders INT DEFAULT 0,
    rating DECIMAL(3,2) DEFAULT 0.00,
    reviews_count INT DEFAULT 0,
    created_at TIMESTAMP NULL,
    updated_at TIMESTAMP NULL,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);
```

### 5. Товари

#### products
```sql
CREATE TABLE products (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    seller_id BIGINT UNSIGNED NULL,
    main_category_id BIGINT UNSIGNED NOT NULL,
    sku VARCHAR(100) NULL,
    model VARCHAR(100) NULL,
    price DECIMAL(15,2) NOT NULL,
    special_price DECIMAL(15,2) NULL,
    special_price_from TIMESTAMP NULL,
    special_price_to TIMESTAMP NULL,
    quantity INT DEFAULT 0,
    min_quantity INT DEFAULT 1,
    stock_status_id BIGINT UNSIGNED NULL,
    status BOOLEAN DEFAULT TRUE,
    featured BOOLEAN DEFAULT FALSE,
    sort_order INT DEFAULT 0,
    views_count INT DEFAULT 0,
    sales_count INT DEFAULT 0,
    created_at TIMESTAMP NULL,
    updated_at TIMESTAMP NULL,
    FOREIGN KEY (seller_id) REFERENCES sellers(id) ON DELETE SET NULL,
    FOREIGN KEY (main_category_id) REFERENCES categories(id) ON DELETE RESTRICT,
    FOREIGN KEY (stock_status_id) REFERENCES stock_statuses(id) ON DELETE SET NULL
);
```

#### product_descriptions
```sql
CREATE TABLE product_descriptions (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    product_id BIGINT UNSIGNED NOT NULL,
    language_id BIGINT UNSIGNED NOT NULL,
    name VARCHAR(255) NOT NULL,
    slug VARCHAR(255) NOT NULL,
    description TEXT NULL,
    short_description TEXT NULL,
    meta_title VARCHAR(255) NULL,
    meta_description TEXT NULL,
    meta_keywords VARCHAR(255) NULL,
    created_at TIMESTAMP NULL,
    updated_at TIMESTAMP NULL,
    FOREIGN KEY (product_id) REFERENCES products(id) ON DELETE CASCADE,
    FOREIGN KEY (language_id) REFERENCES languages(id) ON DELETE CASCADE,
    UNIQUE KEY product_language_unique (product_id, language_id)
);
```

#### product_categories
```sql
CREATE TABLE product_categories (
    product_id BIGINT UNSIGNED NOT NULL,
    category_id BIGINT UNSIGNED NOT NULL,
    PRIMARY KEY (product_id, category_id),
    FOREIGN KEY (product_id) REFERENCES products(id) ON DELETE CASCADE,
    FOREIGN KEY (category_id) REFERENCES categories(id) ON DELETE CASCADE
);
```

#### product_images
```sql
CREATE TABLE product_images (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    product_id BIGINT UNSIGNED NOT NULL,
    image VARCHAR(255) NOT NULL,
    alt_text VARCHAR(255) NULL,
    sort_order INT DEFAULT 0,
    is_main BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP NULL,
    updated_at TIMESTAMP NULL,
    FOREIGN KEY (product_id) REFERENCES products(id) ON DELETE CASCADE
);
```

### 6. Замовлення

#### orders
```sql
CREATE TABLE orders (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    order_number VARCHAR(50) UNIQUE NOT NULL,
    user_id BIGINT UNSIGNED NOT NULL,
    status ENUM('pending', 'processing', 'shipped', 'delivered', 'cancelled', 'refunded') DEFAULT 'pending',
    total DECIMAL(15,2) NOT NULL,
    subtotal DECIMAL(15,2) NOT NULL,
    tax_amount DECIMAL(15,2) DEFAULT 0.00,
    shipping_amount DECIMAL(15,2) DEFAULT 0.00,
    discount_amount DECIMAL(15,2) DEFAULT 0.00,
    payment_method VARCHAR(100) NULL,
    payment_status ENUM('pending', 'paid', 'failed', 'refunded') DEFAULT 'pending',
    shipping_method VARCHAR(100) NULL,
    shipping_address JSON NULL,
    billing_address JSON NULL,
    notes TEXT NULL,
    created_at TIMESTAMP NULL,
    updated_at TIMESTAMP NULL,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE RESTRICT
);
```

#### order_items
```sql
CREATE TABLE order_items (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    order_id BIGINT UNSIGNED NOT NULL,
    product_id BIGINT UNSIGNED NOT NULL,
    seller_id BIGINT UNSIGNED NULL,
    name VARCHAR(255) NOT NULL,
    sku VARCHAR(100) NULL,
    price DECIMAL(15,2) NOT NULL,
    quantity INT NOT NULL,
    total DECIMAL(15,2) NOT NULL,
    created_at TIMESTAMP NULL,
    updated_at TIMESTAMP NULL,
    FOREIGN KEY (order_id) REFERENCES orders(id) ON DELETE CASCADE,
    FOREIGN KEY (product_id) REFERENCES products(id) ON DELETE RESTRICT,
    FOREIGN KEY (seller_id) REFERENCES sellers(id) ON DELETE SET NULL
);
```

### 7. Кошик

#### cart_items
```sql
CREATE TABLE cart_items (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    user_id BIGINT UNSIGNED NULL,
    session_id VARCHAR(255) NULL,
    product_id BIGINT UNSIGNED NOT NULL,
    quantity INT NOT NULL,
    price DECIMAL(15,2) NOT NULL,
    created_at TIMESTAMP NULL,
    updated_at TIMESTAMP NULL,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (product_id) REFERENCES products(id) ON DELETE CASCADE
);
```

### 8. Відгуки

#### reviews
```sql
CREATE TABLE reviews (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    user_id BIGINT UNSIGNED NOT NULL,
    product_id BIGINT UNSIGNED NULL,
    seller_id BIGINT UNSIGNED NULL,
    order_id BIGINT UNSIGNED NULL,
    rating INT NOT NULL CHECK (rating >= 1 AND rating <= 5),
    title VARCHAR(255) NULL,
    comment TEXT NULL,
    status ENUM('pending', 'approved', 'rejected') DEFAULT 'pending',
    created_at TIMESTAMP NULL,
    updated_at TIMESTAMP NULL,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (product_id) REFERENCES products(id) ON DELETE CASCADE,
    FOREIGN KEY (seller_id) REFERENCES sellers(id) ON DELETE CASCADE,
    FOREIGN KEY (order_id) REFERENCES orders(id) ON DELETE SET NULL
);
```

### 9. Блог/Новини

#### articles
```sql
CREATE TABLE articles (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    category_id BIGINT UNSIGNED NULL,
    author_id BIGINT UNSIGNED NOT NULL,
    status ENUM('draft', 'published', 'archived') DEFAULT 'draft',
    featured BOOLEAN DEFAULT FALSE,
    views_count INT DEFAULT 0,
    published_at TIMESTAMP NULL,
    created_at TIMESTAMP NULL,
    updated_at TIMESTAMP NULL,
    FOREIGN KEY (category_id) REFERENCES article_categories(id) ON DELETE SET NULL,
    FOREIGN KEY (author_id) REFERENCES users(id) ON DELETE RESTRICT
);
```

#### article_descriptions
```sql
CREATE TABLE article_descriptions (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    article_id BIGINT UNSIGNED NOT NULL,
    language_id BIGINT UNSIGNED NOT NULL,
    title VARCHAR(255) NOT NULL,
    slug VARCHAR(255) NOT NULL,
    excerpt TEXT NULL,
    content LONGTEXT NOT NULL,
    meta_title VARCHAR(255) NULL,
    meta_description TEXT NULL,
    meta_keywords VARCHAR(255) NULL,
    created_at TIMESTAMP NULL,
    updated_at TIMESTAMP NULL,
    FOREIGN KEY (article_id) REFERENCES articles(id) ON DELETE CASCADE,
    FOREIGN KEY (language_id) REFERENCES languages(id) ON DELETE CASCADE,
    UNIQUE KEY article_language_unique (article_id, language_id)
);
```

### 10. Акції та знижки

#### promotions
```sql
CREATE TABLE promotions (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    type ENUM('percentage', 'fixed', 'buy_x_get_y') NOT NULL,
    value DECIMAL(10,2) NOT NULL,
    min_order_amount DECIMAL(15,2) NULL,
    max_discount_amount DECIMAL(15,2) NULL,
    usage_limit INT NULL,
    used_count INT DEFAULT 0,
    status BOOLEAN DEFAULT TRUE,
    starts_at TIMESTAMP NULL,
    ends_at TIMESTAMP NULL,
    created_at TIMESTAMP NULL,
    updated_at TIMESTAMP NULL
);
```

#### promotion_descriptions
```sql
CREATE TABLE promotion_descriptions (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    promotion_id BIGINT UNSIGNED NOT NULL,
    language_id BIGINT UNSIGNED NOT NULL,
    name VARCHAR(255) NOT NULL,
    description TEXT NULL,
    created_at TIMESTAMP NULL,
    updated_at TIMESTAMP NULL,
    FOREIGN KEY (promotion_id) REFERENCES promotions(id) ON DELETE CASCADE,
    FOREIGN KEY (language_id) REFERENCES languages(id) ON DELETE CASCADE,
    UNIQUE KEY promotion_language_unique (promotion_id, language_id)
);
```

### 11. Додаткові таблиці

#### stock_statuses
```sql
CREATE TABLE stock_statuses (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    created_at TIMESTAMP NULL,
    updated_at TIMESTAMP NULL
);
```

#### stock_status_descriptions
```sql
CREATE TABLE stock_status_descriptions (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    stock_status_id BIGINT UNSIGNED NOT NULL,
    language_id BIGINT UNSIGNED NOT NULL,
    name VARCHAR(100) NOT NULL,
    created_at TIMESTAMP NULL,
    updated_at TIMESTAMP NULL,
    FOREIGN KEY (stock_status_id) REFERENCES stock_statuses(id) ON DELETE CASCADE,
    FOREIGN KEY (language_id) REFERENCES languages(id) ON DELETE CASCADE,
    UNIQUE KEY stock_status_language_unique (stock_status_id, language_id)
);
```

#### wishlist
```sql
CREATE TABLE wishlist (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    user_id BIGINT UNSIGNED NOT NULL,
    product_id BIGINT UNSIGNED NOT NULL,
    created_at TIMESTAMP NULL,
    updated_at TIMESTAMP NULL,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (product_id) REFERENCES products(id) ON DELETE CASCADE,
    UNIQUE KEY user_product_unique (user_id, product_id)
);
```

#### settings
```sql
CREATE TABLE settings (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    key VARCHAR(255) UNIQUE NOT NULL,
    value TEXT NULL,
    type ENUM('string', 'integer', 'boolean', 'json') DEFAULT 'string',
    group_name VARCHAR(100) NULL,
    created_at TIMESTAMP NULL,
    updated_at TIMESTAMP NULL
);
```

## Індекси для оптимізації

```sql
-- Індекси для швидкого пошуку
CREATE INDEX idx_products_status ON products(status);
CREATE INDEX idx_products_featured ON products(featured);
CREATE INDEX idx_products_price ON products(price);
CREATE INDEX idx_products_created_at ON products(created_at);
CREATE INDEX idx_products_seller_id ON products(seller_id);
CREATE INDEX idx_products_main_category_id ON products(main_category_id);

-- Індекси для категорій
CREATE INDEX idx_categories_parent_id ON categories(parent_id);
CREATE INDEX idx_categories_status ON categories(status);
CREATE INDEX idx_categories_sort_order ON categories(sort_order);

-- Індекси для замовлень
CREATE INDEX idx_orders_user_id ON orders(user_id);
CREATE INDEX idx_orders_status ON orders(status);
CREATE INDEX idx_orders_created_at ON orders(created_at);
CREATE INDEX idx_orders_order_number ON orders(order_number);

-- Індекси для продавців
CREATE INDEX idx_sellers_user_id ON sellers(user_id);
CREATE INDEX idx_sellers_status ON sellers(status);

-- Індекси для описів
CREATE INDEX idx_product_descriptions_slug ON product_descriptions(slug);
CREATE INDEX idx_category_descriptions_slug ON category_descriptions(slug);
CREATE INDEX idx_article_descriptions_slug ON article_descriptions(slug);
```

## Зв'язки між таблицями

### Основні зв'язки:
1. **users** → **sellers** (1:1)
2. **users** → **orders** (1:many)
3. **categories** → **categories** (self-referencing, 1:many)
4. **categories** → **products** (1:many)
5. **products** → **sellers** (many:1)
6. **orders** → **order_items** (1:many)
7. **products** → **product_images** (1:many)
8. **products** → **reviews** (1:many)
9. **sellers** → **reviews** (1:many)

### Багатомовні зв'язки:
- **products** → **product_descriptions** (1:many)
- **categories** → **category_descriptions** (1:many)
- **articles** → **article_descriptions** (1:many)
- **promotions** → **promotion_descriptions** (1:many)

## Міграції Laravel

### Приклад міграції для products
```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up()
    {
        Schema::create('products', function (Blueprint $table) {
            $table->id();
            $table->foreignId('seller_id')->nullable()->constrained()->nullOnDelete();
            $table->foreignId('main_category_id')->constrained()->restrictOnDelete();
            $table->string('sku')->nullable();
            $table->string('model')->nullable();
            $table->decimal('price', 15, 2);
            $table->decimal('special_price', 15, 2)->nullable();
            $table->timestamp('special_price_from')->nullable();
            $table->timestamp('special_price_to')->nullable();
            $table->integer('quantity')->default(0);
            $table->integer('min_quantity')->default(1);
            $table->foreignId('stock_status_id')->nullable()->constrained()->nullOnDelete();
            $table->boolean('status')->default(true);
            $table->boolean('featured')->default(false);
            $table->integer('sort_order')->default(0);
            $table->integer('views_count')->default(0);
            $table->integer('sales_count')->default(0);
            $table->timestamps();
            
            $table->index(['status', 'featured']);
            $table->index(['seller_id', 'status']);
            $table->index(['main_category_id', 'status']);
        });
    }

    public function down()
    {
        Schema::dropIfExists('products');
    }
};
```
