from flask import Flask, render_template, request, redirect, url_for
import sqlite3

app = Flask(__name__)

# Database initialization
def init_db():
    conn = sqlite3.connect('merchstore.db')
    cursor = conn.cursor()

    # Create tables
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS User (
            UserID INTEGER PRIMARY KEY,
            Username TEXT,
            Password TEXT,
            Email TEXT,
            FirstName TEXT,
            LastName TEXT,
            Address TEXT,
            PhoneNumber TEXT
        )
    ''')

    cursor.execute('''
        CREATE TABLE IF NOT EXISTS Category (
            CategoryID INTEGER PRIMARY KEY,
            CategoryName TEXT
        )
    ''')

    cursor.execute('''
        CREATE TABLE IF NOT EXISTS Product (
            ProductID INTEGER PRIMARY KEY,
            Name TEXT,
            Description TEXT,
            Price REAL,
            StockQuantity INTEGER,
            CategoryID INTEGER,
            FOREIGN KEY (CategoryID) REFERENCES Category(CategoryID)
        )
    ''')

    cursor.execute('''
        CREATE TABLE IF NOT EXISTS Order (
            OrderID INTEGER PRIMARY KEY,
            UserID INTEGER,
            OrderDate TEXT,
            TotalAmount REAL,
            FOREIGN KEY (UserID) REFERENCES User(UserID)
        )
    ''')

    cursor.execute('''
        CREATE TABLE IF NOT EXISTS OrderItem (
            OrderItemID INTEGER PRIMARY KEY,
            OrderID INTEGER,
            ProductID INTEGER,
            Quantity INTEGER,
            Subtotal REAL,
            FOREIGN KEY (OrderID) REFERENCES Order(OrderID),
            FOREIGN KEY (ProductID) REFERENCES Product(ProductID)
        )
    ''')

    cursor.execute('''
        CREATE TABLE IF NOT EXISTS Cart (
            CartID INTEGER PRIMARY KEY,
            UserID INTEGER,
            FOREIGN KEY (UserID) REFERENCES User(UserID)
        )
    ''')

    cursor.execute('''
        CREATE TABLE IF NOT EXISTS CartProduct (
            CartProductID INTEGER PRIMARY KEY,
            CartID INTEGER,
            ProductID INTEGER,
            Quantity INTEGER,
            FOREIGN KEY (CartID) REFERENCES Cart(CartID),
            FOREIGN KEY (ProductID) REFERENCES Product(ProductID)
        )
    ''')

    conn.commit()
    conn.close()

# Initialize the database
init_db()

# Sample data insertion (for demonstration purposes)
conn = sqlite3.connect('merchstore.db')
cursor = conn.cursor()

cursor.execute('INSERT INTO User (Username, Password, Email, FirstName, LastName, Address, PhoneNumber) VALUES (?, ?, ?, ?, ?, ?, ?)',
               ('user1', 'password1', 'user1@example.com', 'John', 'Doe', '123 Main St', '555-1234'))

cursor.execute('INSERT INTO Category (CategoryName) VALUES (?)', ('Clothing',))
cursor.execute('INSERT INTO Product (Name, Description, Price, StockQuantity, CategoryID) VALUES (?, ?, ?, ?, ?)',
               ('T-Shirt', 'Comfortable cotton T-shirt', 19.99, 100, 1))

cursor.execute('INSERT INTO Cart (UserID) VALUES (?)', (1,))
cursor.execute('INSERT INTO CartProduct (CartID, ProductID, Quantity) VALUES (?, ?, ?)', (1, 1, 2))

conn.commit()
conn.close()

# Route to display products
@app.route('/')
def show_products():
    conn = sqlite3.connect('merchstore.db')
    cursor = conn.cursor()

    cursor.execute('SELECT ProductID, Name, Description, Price FROM Product')
    products = cursor.fetchall()

    conn.close()

    return render_template('products.html', products=products)

# Route to add product to cart
@app.route('/add_to_cart/<int:product_id>')
def add_to_cart(product_id):
    conn = sqlite3.connect('merchstore.db')
    cursor = conn.cursor()

    # Check if the product exists
    cursor.execute('SELECT * FROM Product WHERE ProductID = ?', (product_id,))
    product = cursor.fetchone()

    if product:
        # Check if the user has a cart
        cursor.execute('SELECT * FROM Cart WHERE UserID = ?', (1,))  # Assuming user with ID 1 for simplicity
        cart = cursor.fetchone()

        if not cart:
            # If the user doesn't have a cart, create one
            cursor.execute('INSERT INTO Cart (UserID) VALUES (?)', (1,))  # Assuming user with ID 1 for simplicity

        # Add the product to the cart
        cursor.execute('INSERT INTO CartProduct (CartID, ProductID, Quantity) VALUES (?, ?, ?)', (1, product_id, 1))

        conn.commit()

    conn.close()

    return redirect(url_for('show_products'))

if __name__ == '__main__':
    app.run(debug=True)
