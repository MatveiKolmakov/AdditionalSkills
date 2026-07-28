# SQL Examples — Data Validation for QA

Sample SQL queries demonstrating how SQL is used in manual QA to validate data directly in the database

## Schema Used

```sql
users (id, name, email, created_at)
orders (id, user_id, status, total, created_at)
order_items (id, order_id, product_id, quantity)
products (id, name, price)
```

---

## 1. Verify a new user was created correctly after registration

**QA use case:** After testing the registration flow through the UI, confirm the record actually landed in the database with correct data (no duplicate accounts, correct email format saved).

```sql
SELECT id, name, email, created_at
FROM users
WHERE email = 'test.user@example.com';
```

## 2. Check for duplicate accounts (data integrity)

**QA use case:** Registration form should prevent duplicate emails — verify at the DB level, not just via the UI error message.

```sql
SELECT email, COUNT(*) AS duplicate_count
FROM users
GROUP BY email
HAVING COUNT(*) > 1;
```

## 3. Verify order total matches sum of order items (business logic check)

**QA use case:** After completing checkout, confirm the stored order total actually matches the sum of items — catches backend calculation bugs that wouldn't be visible in the UI alone.

```sql
SELECT o.id AS order_id, o.total AS stored_total,
       SUM(oi.quantity * p.price) AS calculated_total
FROM orders o
JOIN order_items oi ON o.id = oi.order_id
JOIN products p ON oi.product_id = p.id
GROUP BY o.id, o.total
HAVING o.total <> SUM(oi.quantity * p.price);
```
An empty result set means totals are consistent — any returned rows indicate a mismatch bug.

## 4. Find users with more than N orders (test data setup / analysis)

**QA use case:** Useful for finding suitable test accounts (e.g. a "power user" account) or validating reporting/analytics features.

```sql
SELECT u.name, COUNT(o.id) AS order_count
FROM users u
JOIN orders o ON u.id = o.user_id
GROUP BY u.id, u.name
HAVING COUNT(o.id) > 3
ORDER BY order_count DESC;
```

## 5. Check for orphaned records (data cleanup / negative testing follow-up)

**QA use case:** After testing "delete user" or "cancel order" flows, confirm related records were properly cleaned up (or intentionally retained), not left as orphaned data.

```sql
SELECT oi.*
FROM order_items oi
LEFT JOIN orders o ON oi.order_id = o.id
WHERE o.id IS NULL;
```

---

## Notes

These queries are illustrative. written against a hypothetical schema to demonstrate SQL skills relevant to QA (data validation, integrity checks, business logic verification), since SauceDemo itself is a frontend-only demo with no accessible backend/database.
