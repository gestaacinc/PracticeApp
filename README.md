PracticeApp

# Step-by-Step Guide to Building a Frontend Store with Modal Checkout

## Prerequisites

Ensure you have the following installed and set up:

1. **Windows OS**
2. **XAMPP** (for PHP and MySQL):
   - Download and install from [https://www.apachefriends.org/](https://www.apachefriends.org/).
3. **Text Editor or IDE**:
   - Recommended: Visual Studio Code or Sublime Text.
4. **Basic Knowledge**:
   - HTML, CSS, JavaScript, PHP, and MySQL.
5. **Browser**:
   - Google Chrome or any other modern browser.

---

## Step 1: Create the SQL Schema

### 1. Start MySQL in XAMPP
- Open XAMPP Control Panel.
- Start `Apache` and `MySQL`.

### 2. Create a Database
- Open phpMyAdmin: [http://localhost/phpmyadmin](http://localhost/phpmyadmin).
- Create a new database named `store`.

### 3. Create the Tables

Run the following SQL script in phpMyAdmin:

```sql
CREATE TABLE products (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    price DECIMAL(10, 2) NOT NULL,
    stock INT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE orders (
    id INT AUTO_INCREMENT PRIMARY KEY,
    product_id INT NOT NULL,
    quantity INT NOT NULL,
    total_price DECIMAL(10, 2) NOT NULL,
    order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (product_id) REFERENCES products(id)
);

-- Insert sample products
INSERT INTO products (name, price, stock) VALUES
('Product A', 100.00, 50),
('Product B', 150.00, 30),
('Product C', 200.00, 20);
```

---

## Step 2: Project Structure

Create the following directory structure:

```
store/
├── index.php        # Homepage listing products
├── order_process.php # Backend to process orders
├── db.php           # Database connection
├── assets/
│   ├── css/
│   │   └── style.css
│   └── js/
│       └── script.js
└── templates/
    └── modal.php    # Reusable modal form
```

---

## Step 3: Database Connection (db.php)

Create a `db.php` file to handle the database connection:

```php
<?php
$host = 'localhost';
$user = 'root';
$password = ''; // Default password for XAMPP
$dbname = 'store';

$conn = new mysqli($host, $user, $password, $dbname);

if ($conn->connect_error) {
    die("Connection failed: " . $conn->connect_error);
}
?>
```

---

## Step 4: Create the Homepage (index.php)

Display products on the homepage:

```php
<?php
include 'db.php';

// Fetch products from the database
$sql = "SELECT * FROM products";
$result = $conn->query($sql);
?>

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Store</title>
    <link rel="stylesheet" href="assets/css/style.css">
    <script src="assets/js/script.js" defer></script>
</head>
<body>
    <h1>Product List</h1>
    <div class="product-list">
        <?php while ($row = $result->fetch_assoc()): ?>
            <div class="product">
                <h2><?php echo $row['name']; ?></h2>
                <p>Price: PHP <?php echo $row['price']; ?></p>
                <p>Stock: <?php echo $row['stock']; ?></p>
                <button onclick="openModal(<?php echo $row['id']; ?>, '<?php echo $row['name']; ?>', <?php echo $row['price']; ?>)">Order Now</button>
            </div>
        <?php endwhile; ?>
    </div>

    <?php include 'templates/modal.php'; ?>
</body>
</html>
```

---

## Step 5: Create the Modal Form (modal.php)

Reusable modal form for checkout:

```php
<div id="modal" class="modal">
    <div class="modal-content">
        <span class="close" onclick="closeModal()">&times;</span>
        <h2 id="product-name"></h2>
        <form action="order_process.php" method="POST">
            <input type="hidden" name="product_id" id="product-id">
            <label for="quantity">Quantity:</label>
            <input type="number" name="quantity" id="quantity" min="1" required>
            <input type="hidden" name="price" id="product-price">
            <button type="submit">Submit Order</button>
        </form>
    </div>
</div>
```

---

## Step 6: Modal Styling and Script

### 1. CSS (style.css):

```css
body {
    font-family: Arial, sans-serif;
}

.product-list {
    display: flex;
    flex-wrap: wrap;
    gap: 20px;
}

.product {
    border: 1px solid #ddd;
    padding: 10px;
    width: 200px;
}

.modal {
    display: none;
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: rgba(0, 0, 0, 0.5);
    justify-content: center;
    align-items: center;
}

.modal-content {
    background: #fff;
    padding: 20px;
    border-radius: 5px;
}

.close {
    cursor: pointer;
}
```

### 2. JavaScript (script.js):

```javascript
function openModal(id, name, price) {
    document.getElementById('modal').style.display = 'flex';
    document.getElementById('product-id').value = id;
    document.getElementById('product-name').innerText = name;
    document.getElementById('product-price').value = price;
}

function closeModal() {
    document.getElementById('modal').style.display = 'none';
}
```

---

## Step 7: Order Processing (order_process.php)

Handle the order submission:

```php
<?php
include 'db.php';

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $product_id = $_POST['product_id'];
    $quantity = $_POST['quantity'];
    $price = $_POST['price'];
    $total_price = $quantity * $price;

    // Insert order into database
    $sql = "INSERT INTO orders (product_id, quantity, total_price) VALUES (?, ?, ?)";
    $stmt = $conn->prepare($sql);
    $stmt->bind_param('iid', $product_id, $quantity, $total_price);

    if ($stmt->execute()) {
        echo "<p>Order placed successfully!</p>";
    } else {
        echo "<p>Error: " . $stmt->error . "</p>";
    }

    $stmt->close();
}

$conn->close();
?>
```

---

## Step 8: Test the Application

1. Start XAMPP and ensure `Apache` and `MySQL` are running.
2. Place the `store` folder inside `htdocs` (e.g., `C:\xampp\htdocs\store`).
3. Access the application in your browser: [http://localhost/store/index.php](http://localhost/store/index.php).
4. Add sample products to the `products` table in phpMyAdmin and test the checkout process.

---

 
