
# task5
import tkinter as tk
from tkinter import ttk, messagebox
import sqlite3
import os
from datetime import datetime

# Create a database to store product and customer details
conn = sqlite3.connect('billing.db')
cursor = conn.cursor()

cursor.execute('''
CREATE TABLE IF NOT EXISTS products (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    price REAL NOT NULL
)
''')

cursor.execute('''
CREATE TABLE IF NOT EXISTS customers (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    contact TEXT NOT NULL
)
''')

conn.commit()

class BillingApp:
    def __init__(self, root):
        self.root = root
        self.root.title("Billing Software")
        self.root.geometry("800x600")

        self.product_frame = ttk.LabelFrame(root, text="Products")
        self.product_frame.grid(row=0, column=0, padx=10, pady=10, sticky="ew")

        self.customer_frame = ttk.LabelFrame(root, text="Customer Details")
        self.customer_frame.grid(row=0, column=1, padx=10, pady=10, sticky="ew")

        self.invoice_frame = ttk.LabelFrame(root, text="Invoice")
        self.invoice_frame.grid(row=1, column=0, columnspan=2, padx=10, pady=10, sticky="nsew")

        # Product Frame Widgets
        ttk.Label(self.product_frame, text="Product Name:").grid(row=0, column=0, padx=5, pady=5, sticky="e")
        self.product_name_var = tk.StringVar()
        ttk.Entry(self.product_frame, textvariable=self.product_name_var).grid(row=0, column=1, padx=5, pady=5, sticky="w")

        ttk.Label(self.product_frame, text="Price:").grid(row=1, column=0, padx=5, pady=5, sticky="e")
        self.product_price_var = tk.DoubleVar()
        ttk.Entry(self.product_frame, textvariable=self.product_price_var).grid(row=1, column=1, padx=5, pady=5, sticky="w")

        ttk.Button(self.product_frame, text="Add Product", command=self.add_product).grid(row=2, column=0, columnspan=2, pady=5)

        # Customer Frame Widgets
        ttk.Label(self.customer_frame, text="Customer Name:").grid(row=0, column=0, padx=5, pady=5, sticky="e")
        self.customer_name_var = tk.StringVar()
        ttk.Entry(self.customer_frame, textvariable=self.customer_name_var).grid(row=0, column=1, padx=5, pady=5, sticky="w")

        ttk.Label(self.customer_frame, text="Contact:").grid(row=1, column=0, padx=5, pady=5, sticky="e")
        self.customer_contact_var = tk.StringVar()
        ttk.Entry(self.customer_frame, textvariable=self.customer_contact_var).grid(row=1, column=1, padx=5, pady=5, sticky="w")

        ttk.Button(self.customer_frame, text="Add Customer", command=self.add_customer).grid(row=2, column=0, columnspan=2, pady=5)

        # Invoice Frame Widgets
        self.invoice_text = tk.Text(self.invoice_frame, wrap="word", width=70, height=20)
        self.invoice_text.pack(padx=5, pady=5)

        ttk.Button(root, text="Generate Invoice", command=self.generate_invoice).pack(pady=5)

    def add_product(self):
        name = self.product_name_var.get()
        price = self.product_price_var.get()

        if name and price:
            cursor.execute('INSERT INTO products (name, price) VALUES (?, ?)', (name, price))
            conn.commit()
            messagebox.showinfo("Success", "Product added successfully!")
        else:
            messagebox.showwarning("Input Error", "Please enter both product name and price.")

    def add_customer(self):
        name = self.customer_name_var.get()
        contact = self.customer_contact_var.get()

        if name and contact:
            cursor.execute('INSERT INTO customers (name, contact) VALUES (?, ?)', (name, contact))
            conn.commit()
            messagebox.showinfo("Success", "Customer added successfully!")
        else:
            messagebox.showwarning("Input Error", "Please enter both customer name and contact.")

    def generate_invoice(self):
        self.invoice_text.delete('1.0', tk.END)
        customer_name = self.customer_name_var.get()
        customer_contact = self.customer_contact_var.get()

        if customer_name and customer_contact:
            self.invoice_text.insert(tk.END, f"Customer: {customer_name}\n")
            self.invoice_text.insert(tk.END, f"Contact: {customer_contact}\n")
            self.invoice_text.insert(tk.END, "\nProducts:\n")

            cursor.execute('SELECT * FROM products')
            products = cursor.fetchall()

            total = 0
            for product in products:
                self.invoice_text.insert(tk.END, f"{product[1]} - ${product[2]:.2f}\n")
                total += product[2]

            self.invoice_text.insert(tk.END, f"\nTotal: ${total:.2f}\n")
            self.invoice_text.insert(tk.END, f"Date: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}\n
