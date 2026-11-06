# SS-01: ShopSphere Schema Documentation

**Assignment ID:** SS-01
**Topic:** Database Fundamentals
**Database:** shopsphere_analytics 

## Business Requirement
A new analyst is joining the team. Create a data guide explaining
how the ShopSphere database is organised.

## Task 1: Table Summary

| Table       | Primary Key   | Grain (one row = …)              | Business use                                                          |
|-------------|---------------|----------------------------------|-----------------------------------------------------------------------|
| customers   | customer_id   | one customer                     | Who are our customers and where are they located?                     |
| products    | product_id    | one product                      | What products do we have, and which have the highest profit per unit? |
| categories  | category_id   | one category                     | Which categories have the most expensive products?                    |
| orders      | order_id      | one order                        | How many orders were placed today?                                    |
| order_items | order_item_id | one product line within an order | Which products sell the most?                                         |
| payments    | payment_id    | one payment                      | Which payment method do customers use most?                           |

## Task 2: Relationships

| Foreign key            | Points to  | Relationship                                           |
|------------------------|------------|--------------------------------------------------------|
| products.category_id   | categories | One category can have many products                    |
| orders.customer_id     | customers  | One customer can place many orders                     |
| order_items.order_id   | orders     | One order can contain many products (lines)            |
| order_items.product_id | products   | One product can appear in many orders                  |
| payments.order_id      | orders     | One order can have many payments (e.g. failed + retry) |

```
customers ◄── orders ◄── order_items ──► products ──► categories
                ▲
                └──── payments
```

## Task 3: Data Quality Notes

### Nullable columns

| Column                  | Nullable? |
|-------------------------|-----------|
| customers.email         | YES       |
| customers.phone         | YES       |
| customers.city          | YES       |
| customers.state         | YES       |
| customers.signup_date   | YES       |
| payments.payment_method | YES       |
| payments.payment_status | YES       |
| payments.payment_date   | YES       |

`products`, `categories`, `orders` and `order_items` have no nullable columns,
so the core sales data is complete. The risk areas are customer details and
payment details.

### Column notes

**payments.payment_date**
- Why it can be NULL: for COD orders, the customer hasn't paid yet,
  so the payment date is unknown when the record is created.
- Analyst caution: "Money received in January" must count only payments
  that actually happened. Unpaid COD orders have a NULL date, so they
  won't appear in a date-based report. Revenue from orders placed
  is NOT the same as money received.

**payments.payment_method**
- Why it can be NULL: the customer closed the site before choosing a
  payment method, so ShopSphere doesn't know how they will pay.
- Analyst caution: when calculating "% of payments by method", the result
  changes depending on whether NULL rows are included or excluded.
  Always state which approach was used.

## Task 4: Why order_items is a separate table

`order_items` exists as a separate table because one order can contain many
products. If we stored products inside `orders`, we wouldn't know how many
columns we’d need, and most of them would be NULL.


## Learning Notes

1. I learned that NULL means the value is missing or unknown.
for example:
   payment_date is NULL when a COD customer hasn't paid yet.

2. I learned that a primary key is important because it uniquely
   identifies each row, unlike customer_name, which can be the same
   for two different customers.
   
3. A mistake I made was writing column names with spaces (order id) instead
   of underscores (order_id), and now I know SQL reads a space as two separate
   words, so it causes an error. I must use the exact column name.
