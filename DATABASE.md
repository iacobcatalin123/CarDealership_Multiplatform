# CarDealershipV2 Database Documentation

This document provides detailed information about the database structure, relationships, and common queries used in the CarDealershipV2 system.

## Schema Overview

### Tables

#### 1. dealership_inventory
Stores all vehicle information and stock levels.

```sql
CREATE TABLE IF NOT EXISTS `dealership_inventory` (
    `id` int(11) NOT NULL AUTO_INCREMENT,
    `model` varchar(64) NOT NULL,
    `name` varchar(128) NOT NULL,
    `price` int(11) NOT NULL,
    `category` varchar(50) NOT NULL DEFAULT 'Cars',
    `stock` int(11) NOT NULL DEFAULT 0,
    `is_used` tinyint(1) NOT NULL DEFAULT 0,
    `mileage` int(11) DEFAULT NULL,
    `description` text,
    `specs` json,
    PRIMARY KEY (`id`),
    KEY `model` (`model`),
    KEY `category` (`category`)
);
```

Fields:
- `id`: Unique identifier
- `model`: Vehicle spawn code
- `name`: Display name
- `price`: Current price
- `category`: Vehicle category (Cars, Used, VIP)
- `stock`: Available quantity
- `is_used`: Used vehicle flag
- `mileage`: For used vehicles
- `description`: Vehicle description
- `specs`: JSON object with specifications

#### 2. dealership_sales
Tracks all vehicle sales and transactions.

```sql
CREATE TABLE IF NOT EXISTS `dealership_sales` (
    `id` int(11) NOT NULL AUTO_INCREMENT,
    `inventory_id` int(11) NOT NULL,
    `buyer_id` int(11) NOT NULL,
    `price_paid` int(11) NOT NULL,
    `purchase_date` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
    `plate` varchar(8) NOT NULL,
    `vin` varchar(17) NOT NULL,
    PRIMARY KEY (`id`),
    KEY `inventory_id` (`inventory_id`),
    KEY `buyer_id` (`buyer_id`),
    KEY `plate` (`plate`),
    KEY `vin` (`vin`)
);
```

Fields:
- `id`: Unique identifier
- `inventory_id`: Reference to dealership_inventory
- `buyer_id`: Player character ID
- `price_paid`: Final sale price
- `purchase_date`: Transaction timestamp
- `plate`: Vehicle license plate
- `vin`: Vehicle identification number

## Common Queries

### 1. Vehicle Inventory Management

#### Get Available Vehicles
```sql
SELECT * FROM dealership_inventory 
WHERE stock > 0 AND category = ? 
ORDER BY price ASC;
```

#### Update Stock Level
```sql
UPDATE dealership_inventory 
SET stock = GREATEST(0, stock + ?) 
WHERE id = ?;
```

#### Add New Vehicle
```sql
INSERT INTO dealership_inventory 
(model, name, price, category, stock, description, specs) 
VALUES (?, ?, ?, ?, ?, ?, ?);
```

#### Create Used Vehicle
```sql
INSERT INTO dealership_inventory 
(model, name, price, category, stock, is_used, mileage, description, specs)
SELECT 
    model, 
    CONCAT(name, ' (Used)'),
    ROUND(price * 0.7), -- 30% discount
    'Used',
    1,
    1,
    ?,
    description,
    specs
FROM dealership_inventory 
WHERE id = ?;
```

### 2. Sales Tracking

#### Record Sale
```sql
INSERT INTO dealership_sales 
(inventory_id, buyer_id, price_paid, plate, vin) 
VALUES (?, ?, ?, ?, ?);
```

#### Get Sales History
```sql
SELECT 
    s.id, 
    s.price_paid,
    s.purchase_date,
    s.plate,
    i.name as vehicle_name,
    i.model,
    i.category
FROM dealership_sales s
JOIN dealership_inventory i ON s.inventory_id = i.id
WHERE s.purchase_date BETWEEN ? AND ?
ORDER BY s.purchase_date DESC;
```

#### Sales Summary by Category
```sql
SELECT 
    i.category,
    COUNT(*) as total_sales,
    SUM(s.price_paid) as total_revenue
FROM dealership_sales s
JOIN dealership_inventory i ON s.inventory_id = i.id
GROUP BY i.category;
```

### 3. Vehicle Validation

#### Check Plate Availability
```sql
SELECT 1 
FROM (
    SELECT plate FROM vehicles 
    UNION 
    SELECT plate FROM dealership_sales
) AS plates 
WHERE plate = ?;
```

#### Check VIN Availability
```sql
SELECT 1 
FROM (
    SELECT vin FROM vehicles 
    UNION 
    SELECT vin FROM dealership_sales
) AS vins 
WHERE vin = ?;
```

### 4. Stock Management

#### Low Stock Alert
```sql
SELECT id, model, name, stock 
FROM dealership_inventory 
WHERE stock <= 3 
ORDER BY stock ASC;
```

#### Popular Vehicles
```sql
SELECT 
    i.id,
    i.name,
    COUNT(*) as sales_count
FROM dealership_sales s
JOIN dealership_inventory i ON s.inventory_id = i.id
WHERE s.purchase_date >= DATE_SUB(NOW(), INTERVAL 30 DAY)
GROUP BY i.id
ORDER BY sales_count DESC
LIMIT 10;
```

## Transactions

### Purchase Transaction
```sql
-- Example of a complete purchase transaction
START TRANSACTION;

-- 1. Check and update stock
UPDATE dealership_inventory 
SET stock = stock - 1 
WHERE id = ? AND stock > 0;

-- 2. Record sale
INSERT INTO dealership_sales 
(inventory_id, buyer_id, price_paid, plate, vin) 
VALUES (?, ?, ?, ?, ?);

-- 3. Add to vehicles table
INSERT INTO vehicles 
(plate, vin, model, owner, stored, category, price) 
VALUES (?, ?, ?, ?, 'Dealership', ?, ?);

COMMIT;
```

## Maintenance Queries

### 1. Cleanup
```sql
-- Remove old test drive records
DELETE FROM test_drives 
WHERE end_time < DATE_SUB(NOW(), INTERVAL 1 DAY);

-- Archive old sales
INSERT INTO dealership_sales_archive 
SELECT * FROM dealership_sales 
WHERE purchase_date < DATE_SUB(NOW(), INTERVAL 1 YEAR);
```

### 2. Optimization
```sql
-- Analyze and optimize tables
ANALYZE TABLE dealership_inventory, dealership_sales;
OPTIMIZE TABLE dealership_inventory, dealership_sales;
```

### 3. Backup
```sql
-- Backup tables
CREATE TABLE dealership_inventory_backup AS SELECT * FROM dealership_inventory;
CREATE TABLE dealership_sales_backup AS SELECT * FROM dealership_sales;
```

## Performance Considerations

1. **Indexing**
   - Primary keys on all tables
   - Indexes on frequently searched columns
   - Composite indexes for common queries

2. **Query Optimization**
   - Use prepared statements
   - Limit result sets
   - Use appropriate JOIN types
   - Avoid SELECT *

3. **Maintenance**
   - Regular backups
   - Index optimization
   - Table analysis
   - Query monitoring

## Security

1. **Access Control**
   - Use prepared statements
   - Validate all input
   - Proper permissions
   - Transaction handling

2. **Data Integrity**
   - Foreign key constraints
   - Input validation
   - Transaction management
   - Error handling

## Backup Strategy

1. **Regular Backups**
   ```sql
   mysqldump -u username -p database_name dealership_inventory dealership_sales > backup.sql
   ```

2. **Automated Backups**
   - Daily incremental
   - Weekly full backup
   - Monthly archives
   - Yearly consolidation

## Error Handling

1. **Common Errors**
   - Duplicate keys
   - Foreign key violations
   - Transaction deadlocks
   - Connection issues

2. **Recovery Procedures**
   - Transaction rollback
   - Data verification
   - Integrity checks
   - Error logging

## Best Practices

1. **Database Design**
   - Normalize tables
   - Use appropriate data types
   - Implement constraints
   - Document changes

2. **Query Optimization**
   - Use indexes effectively
   - Optimize JOIN operations
   - Limit result sets
   - Monitor performance

3. **Maintenance**
   - Regular backups
   - Index optimization
   - Query monitoring
   - Error logging