# Real Time Data Synchronization using Publication and Subscription in PostgreSQL 🚀
Logical Replication in PostgreSQL allows selective replication of data between databases, offering fine-grained control over what is replicated. Unlike streaming replication, which replicates the entire database, it enables replication of specific tables, schemas, or rows based on custom conditions.

### Enable Logical Replication
```bash
wal_level = logical
max_replication_slots = 7  -- or more, depending on your needs
max_replication_workers = 4 -- or more, depending on your needs
```
### Create a Publisher Database
- On the Publisher Server:
```sql
CREATE DATABASE publisher_db;
```

- 🔌 Connect to the database:
```sql
\c publisher_db
```

### Create a Table & Insert Sample Data
```sql
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name TEXT,
    price NUMERIC
);
INSERT INTO products (name, price) VALUES ('Laptop', 1200), ('Mouse', 30);
```


### Create a Publication 📢

```sql
CREATE PUBLICATION publication_for_products FOR TABLE products;
```
- Creates a publication named `publication_for_products`.
- It will publish changes (insert, update, delete) for the products table to subscribers.

- Check if the publication exists
```sql
SELECT * FROM pg_publication;
```

- Create a replication user and set a password 🔐
```sql
CREATE ROLE anower_replicator WITH REPLICATION LOGIN PASSWORD 'anower_replicator123';
GRANT ALL PRIVILEGES ON DATABASE publisher_db TO anower_replicator;
GRANT SELECT ON products TO anower_replicator;
```
- Creates a role anower_replicator with REPLICATION and LOGIN privileges, allowing it to log in and participate in logical replication.
- Grants full access to publisher_db, allowing the role to connect, create, and manage objects.

- Allow Remote Connections

```nginx
host    replication    anower_replicator    <subscriber_server_ip>/32    scram-sha-256
```

Restart PostgreSQL
```bash
sudo systemctl restart postgresql-16
```


# On the Subscriber Server

### Create a Subscriber Database
```sql
CREATE DATABASE subscriber_db;
```

- 🔌 Connect to the database:
```sql
\c subscriber_db
```

### Create a Subscription
```sql
CREATE SUBSCRIPTION subscription_for_products 
    CONNECTION 'host=<publisher_server_ip> dbname=publisher_db user=anower_replicator password=anower_replicator123'
    PUBLICATION publication_for_products;
```
- This connects the Subscriber (your current database) to the Publisher (publisher_db).
- It tells PostgreSQL to listen for changes from publication_for_products.
- PostgreSQL fetches the current data from publication_for_products.
- It starts tracking updates, inserts, and deletes from the products table.


✅ Check subscription status
```sql
SELECT * FROM pg_subscription;
```
on the Subscriber Server, you will get details about the active subscriptions.
`subenabled` – `t` (true) if the subscription is active.

### Verify Data Sync 🎉
- On the Subscriber Server, check the data:
```sql
SELECT * FROM products;
```

🎉 If everything works fine, you should see the same data as in the publisher!

### 🏁 Final Check

- Try inserting new data on the Publisher Server:
```sql
INSERT INTO products (name, price) VALUES ('Keyboard', 50);
```

- Now, check on the Subscriber Server:
```sql
SELECT * FROM products;
```

✨ Success! The new data should appear automatically. 🎉

### Benefits 

- `Efficient data replication:` Replicates only necessary data.
- `Selective data sharing:` Control over what data is replicated.
- `Real-time updates:` Instant synchronization of changes.
- `System decoupling:` Independent publisher and subscriber systems.
- `Scalability:` Easy to add multiple subscribers.
- `No READ-ONLY mode:` Enable to write on the table.
