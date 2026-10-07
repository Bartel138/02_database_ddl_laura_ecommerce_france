# 02_database_ddl_laura_ecommerce_france
Script SQL de création de la base de données et des 13 tables. Intègre la gestion des types de données optimisés (DECIMAL, TIMESTAMP), le moteur InnoDB et les contraintes d'intégrité référentielle.

# 🛠️ Projet 2 : Création de la Base de Données (DDL)

## 📋 Description du Projet
Script SQL de création de la base de données et des 13 tables pour **Laura E-Commerce France**. Intègre la gestion des types de données optimisés (`DECIMAL`, `TIMESTAMP`), le moteur InnoDB et les contraintes d'intégrité référentielle.

---

## 💻 Script SQL Complet

```sql
-- Création de la base de données avec encodage UTF-8
CREATE DATABASE IF NOT EXISTS laura_ecommerce_france 
CHARACTER SET utf8mb4 
COLLATE utf8mb4_unicode_ci;

USE laura_ecommerce_france;

-- Désactivation temporaire des vérifications de clés étrangères pour l'ordre de création
SET FOREIGN_KEY_CHECKS = 0;

-- 1. Table des Pays (Référentiel international)
CREATE TABLE IF NOT EXISTS country (
    country_id INT AUTO_INCREMENT PRIMARY KEY,
    country_name VARCHAR(100) NOT NULL UNIQUE,
    iso_code VARCHAR(3) NOT NULL UNIQUE
) ENGINE=InnoDB;

-- 2. Table des Devises
CREATE TABLE IF NOT EXISTS currency (
    currency_id INT AUTO_INCREMENT PRIMARY KEY,
    currency_code VARCHAR(3) NOT NULL UNIQUE,
    exchange_rate_to_eur DECIMAL(10,4) NOT NULL
) ENGINE=InnoDB;

-- 3. Table des Clients
CREATE TABLE IF NOT EXISTS customer (
    customer_id INT AUTO_INCREMENT PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(100) NOT NULL UNIQUE,
    phone VARCHAR(20),
    country_id INT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT fk_customer_country FOREIGN KEY (country_id) REFERENCES country(country_id)
) ENGINE=InnoDB;

-- 4. Table des Adresses
CREATE TABLE IF NOT EXISTS address (
    address_id INT AUTO_INCREMENT PRIMARY KEY,
    customer_id INT NOT NULL,
    street_address VARCHAR(255) NOT NULL,
    city VARCHAR(100) NOT NULL,
    postal_code VARCHAR(20) NOT NULL,
    country_id INT NOT NULL,
    CONSTRAINT fk_address_customer FOREIGN KEY (customer_id) REFERENCES customer(customer_id) ON DELETE CASCADE,
    CONSTRAINT fk_address_country FOREIGN KEY (country_id) REFERENCES country(country_id)
) ENGINE=InnoDB;

-- 5. Table des Catégories de Produits
CREATE TABLE IF NOT EXISTS category (
    category_id INT AUTO_INCREMENT PRIMARY KEY,
    category_name VARCHAR(100) NOT NULL UNIQUE,
    parent_category_id INT,
    CONSTRAINT fk_category_parent FOREIGN KEY (parent_category_id) REFERENCES category(category_id)
) ENGINE=InnoDB;

-- 6. Table des Produits
CREATE TABLE IF NOT EXISTS product (
    product_id INT AUTO_INCREMENT PRIMARY KEY,
    product_name VARCHAR(150) NOT NULL,
    category_id INT,
    price DECIMAL(10,2) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT fk_product_category FOREIGN KEY (category_id) REFERENCES category(category_id)
) ENGINE=InnoDB;

-- 7. Table des Entrepôts
CREATE TABLE IF NOT EXISTS warehouse (
    warehouse_id INT AUTO_INCREMENT PRIMARY KEY,
    warehouse_name VARCHAR(100) NOT NULL,
    country_id INT,
    CONSTRAINT fk_warehouse_country FOREIGN KEY (country_id) REFERENCES country(country_id)
) ENGINE=InnoDB;

-- 8. Table des Stocks (Inventaire)
CREATE TABLE IF NOT EXISTS inventory (
    inventory_id INT AUTO_INCREMENT PRIMARY KEY,
    product_id INT NOT NULL,
    warehouse_id INT NOT NULL,
    stock_quantity INT NOT NULL DEFAULT 0,
    CONSTRAINT fk_inventory_product FOREIGN KEY (product_id) REFERENCES product(product_id),
    CONSTRAINT fk_inventory_warehouse FOREIGN KEY (warehouse_id) REFERENCES warehouse(warehouse_id)
) ENGINE=InnoDB;

-- 9. Table des Commandes
CREATE TABLE IF NOT EXISTS orders (
    order_id INT AUTO_INCREMENT PRIMARY KEY,
    customer_id INT NOT NULL,
    order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    order_status VARCHAR(50) NOT NULL,
    total_amount DECIMAL(10,2) NOT NULL,
    CONSTRAINT fk_orders_customer FOREIGN KEY (customer_id) REFERENCES customer(customer_id)
) ENGINE=InnoDB;

-- 10. Table des Lignes de Commandes
CREATE TABLE IF NOT EXISTS order_item (
    order_item_id INT AUTO_INCREMENT PRIMARY KEY,
    order_id INT NOT NULL,
    product_id INT NOT NULL,
    quantity INT NOT NULL,
    unit_price DECIMAL(10,2) NOT NULL,
    CONSTRAINT fk_order_item_order FOREIGN KEY (order_id) REFERENCES orders(order_id) ON DELETE CASCADE,
    CONSTRAINT fk_order_item_product FOREIGN KEY (product_id) REFERENCES product(product_id)
) ENGINE=InnoDB;

-- 11. Table des Paiements
CREATE TABLE IF NOT EXISTS payment (
    payment_id INT AUTO_INCREMENT PRIMARY KEY,
    order_id INT NOT NULL,
    payment_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    payment_method VARCHAR(50) NOT NULL,
    amount DECIMAL(10,2) NOT NULL,
    payment_status VARCHAR(50) NOT NULL,
    CONSTRAINT fk_payment_order FOREIGN KEY (order_id) REFERENCES orders(order_id) ON DELETE CASCADE
) ENGINE=InnoDB;

-- 12. Table de la Logistique (Shipping)
CREATE TABLE IF NOT EXISTS shipping (
    shipping_id INT AUTO_INCREMENT PRIMARY KEY,
    order_id INT NOT NULL,
    carrier_name VARCHAR(100) NOT NULL,
    tracking_number VARCHAR(100) UNIQUE,
    shipping_status VARCHAR(50) NOT NULL,
    shipped_date TIMESTAMP NULL,
    CONSTRAINT fk_shipping_order FOREIGN KEY (order_id) REFERENCES orders(order_id) ON DELETE CASCADE
) ENGINE=InnoDB;

-- 13. Table des Avis Clients (Reviews)
CREATE TABLE IF NOT EXISTS review (
    review_id INT AUTO_INCREMENT PRIMARY KEY,
    product_id INT NOT NULL,
    customer_id INT NOT NULL,
    rating INT CHECK (rating BETWEEN 1 AND 5),
    comment TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT fk_review_product FOREIGN KEY (product_id) REFERENCES product(product_id) ON DELETE CASCADE,
    CONSTRAINT fk_review_customer FOREIGN KEY (customer_id) REFERENCES customer(customer_id) ON DELETE CASCADE
) ENGINE=InnoDB;

-- Réactivation des vérifications de clés étrangères
SET FOREIGN_KEY_CHECKS = 1;
