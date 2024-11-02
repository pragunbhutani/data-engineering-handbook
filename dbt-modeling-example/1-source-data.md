# Source Data

For the sake of our illustration, we will consider the example of a simple B2C ecommerce app where users
can purchase products. Users are served by merchants that store products.

Each product is assigned a category. Merchants acquire these products from suppliers - called "vendors" - and store them
till they are sold to a customer.

## Business Context

### App Workflow

- Users log in and see items available at the nearest merchant.
- Users add items to a virtual shopping cart.
- Users check out the cart as a single order, which is delivered to their location.
- Each order item has a landed cost price and an effective selling price
- Effective selling price is calculated as the base selling price minus any discounts or promotions.

### Key Metrics

These are the key metrics that the business is interested in:

- GMV: Sum of the selling price of all orders.
- Total Margin: Sum of the margin (selling price - landed cost price) of all orders.
- Aggregations by date, merchant and product category.

### Additional Complexity

- Each merchant acquires items from vendors in batches, with varying prices.
- The effective landed cost price is a weighted rolling average recalculated daily.
- Promotions are applied at the item level.

## Source Tables

Let's define the schema of the source tables as they would appear in the production database:

### Users

Contains information about the users of the app.

| Column Name | Type      | Description                         |
| ----------- | --------- | ----------------------------------- |
| id          | VARCHAR   | Primary Key                         |
| name        | VARCHAR   | Name of the user                    |
| email       | VARCHAR   | Email of the user                   |
| lat         | FLOAT     | Latitude of the user                |
| long        | FLOAT     | Longitude of the user               |
| createdAt   | TIMESTAMP | Time when the user was created      |
| updatedAt   | TIMESTAMP | Time when the user was last updated |

### Merchants

Contains information about the merchants who store and sell items to users.

| Column Name   | Type      | Description                             |
| ------------- | --------- | --------------------------------------- |
| merchant_id   | VARCHAR   | Primary Key                             |
| merchant_name | VARCHAR   | Name of the merchant                    |
| geo_loc       | GEOGRAPHY | Latitude/Longitude of the merchant      |
| createdAt     | TIMESTAMP | Time when the merchant was created      |
| updatedAt     | TIMESTAMP | Time when the merchant was last updated |

### Items

Contains information about the items available for sale.

| Column Name | Type      | Description                                            |
| ----------- | --------- | ------------------------------------------------------ |
| item_id     | VARCHAR   | Primary Key                                            |
| merchant_id | VARCHAR   | Foreign Key referencing merchant_id in merchants table |
| name        | VARCHAR   | Name of the item                                       |
| category    | VARCHAR   | Category of the item                                   |
| price       | FLOAT     | Base selling price of the item                         |
| created_at  | TIMESTAMP | Time when the item was created                         |
| updated_at  | TIMESTAMP | Time when the item was last updated                    |

### Promotions

Contains information about the promotions applied to items.

| Column Name  | Type      | Description                                    |
| ------------ | --------- | ---------------------------------------------- |
| promotion_id | VARCHAR   | Primary Key                                    |
| product_id   | VARCHAR   | Name of the item to which promotion is applied |
| discount     | FLOAT     | Percentage discount applied to the item        |
| created_at   | TIMESTAMP | Time when the promotion was created            |
| updated_at   | TIMESTAMP | Time when the promotion was last updated       |

### Inventory

Contains information about the batches of items acquired by merchants.

| Column Name   | Type      | Description                                            |
| ------------- | --------- | ------------------------------------------------------ |
| inventory_id  | SERIAL    | Primary Key                                            |
| merchant_id   | INT       | Foreign Key referencing merchant_id in merchants table |
| item_id       | INT       | Foreign Key referencing item_id in items table         |
| qty           | INT       | Quantity of the item in inventory                      |
| date_acquired | DATE      | Date when the item was acquired                        |
| cost          | DECIMAL   | Cost price of the item                                 |
| created_at    | TIMESTAMP | Time when the inventory record was created             |
| updated_at    | TIMESTAMP | Time when the inventory record was last updated        |

### Orders

Contains information about the orders placed by users.

| Column Name | Type      | Description                                            |
| ----------- | --------- | ------------------------------------------------------ |
| order_id    | SERIAL    | Primary Key                                            |
| customer_id | INT       | Foreign Key referencing user_id in users table         |
| store_id    | INT       | Foreign Key referencing merchant_id in merchants table |
| date        | TIMESTAMP | Date and time of the order                             |
| created_at  | TIMESTAMP | Time when the order was created                        |
| updated_at  | TIMESTAMP | Time when the order was last updated                   |

### Order Items

Contains information about the items in each order.

| Column Name   | Type      | Description                                      |
| ------------- | --------- | ------------------------------------------------ |
| order_item_id | SERIAL    | Primary Key                                      |
| order_id      | INT       | Foreign Key referencing order_id in orders table |
| product_id    | INT       | Foreign Key referencing item_id in items table   |
| qty           | INT       | Quantity of the item in the order                |
| created_at    | TIMESTAMP | Time when the order item was created             |
| updated_at    | TIMESTAMP | Time when the order item was last updated        |

## Next Steps

In this section, we have defined the source tables that form the basis of our data model.

You'll notice that there are some irregularities in the way columns are named, or in terms of how information like
locations are stored. This is quite common in real-world data sources.

In the next section, we will create the staging layer where we can make sure all columns follow
the correct data types and nomenclature.
