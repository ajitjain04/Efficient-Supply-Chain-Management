from flask import Flask, jsonify, request, render_template
from flask_cors import CORS
import mysql.connector
from mysql.connector import Error

app = Flask(__name__, template_folder='/Users/adeshjain/Desktop/adesh_coding/dcdsl_proj/dcdsl/templates')  
CORS(app)

# Database configuration
DB_HOST = 'localhost'
DB_USER = 'root'
DB_DATABASE = 'project_final'

def get_db_connection():
    try:
        conn = mysql.connector.connect(
            host=DB_HOST,
            user=DB_USER,
            database=DB_DATABASE
        )
        if conn.is_connected():
            return conn
        else:
            return None
    except Error as e:
        print(f"Error: {e}")
        return None

# Serve the HTML frontend
@app.route('/')
def index():
    return render_template('index9999.html')  # Ensure 'index69.html' is located in the 'templates' folder

# API route to execute SQL queries
@app.route('/execute_query/<query_id>', methods=['GET'])
def execute_query(query_id):
    query_result = None
    error = None

    # Dictionary of queries
    queries = {
        "query1": "SELECT Product_Name, Product_Price FROM product_info;",
        "query2": "SELECT COUNT(*) AS total_orders FROM orders;",
        "query3": "SELECT p.Product_Name FROM product_info p JOIN category c ON p.Category_Id = c.Id WHERE c.Name = 'Furniture';",
        "query4": "SELECT c.First_Name, c.Last_Name, o.Order_Id FROM customer_info c JOIN orders o ON c.Id = o.Customer_Id;",
        "query5": "SELECT DISTINCT c.First_Name, c.Last_Name FROM customer_info c JOIN orders o ON c.Id = o.Customer_Id WHERE o.Order_Date >= DATE_SUB(CURDATE(), INTERVAL 1 MONTH);",
        "query6": "SELECT oi.Item_Id, SUM(oi.Sales) AS total_sales FROM ordered_items oi GROUP BY oi.Item_Id;",
        "query7": """
            SELECT c.Name AS category_name, COUNT(p.Product_Id) AS product_count
            FROM category c
            LEFT JOIN product_info p ON c.Id = p.Category_Id
            GROUP BY c.Name;
        """,
        "query8": "SELECT * FROM active_customers;",
        "query9": "SELECT p.Product_Name, SUM(oi.Sales) AS total_sales FROM product_info p JOIN ordered_items oi ON p.Product_Id = oi.Item_Id GROUP BY p.Product_Name ORDER BY total_sales DESC LIMIT 5;",
        "query10": "SELECT o.Order_Id, SUM(oi.Sales) AS total_order_value FROM orders o JOIN ordered_items oi ON o.Order_Id = oi.Order_Id GROUP BY o.Order_Id;",
        "query11": "SELECT o.Order_State, AVG(o.Real_Shipping_Days) AS Avg_Real_Shipping_Days, AVG(o.Scheduled_Shipping_Days) AS Avg_Scheduled_Shipping_Days FROM orders o GROUP BY o.Order_State ORDER BY Avg_Real_Shipping_Days DESC;",
        "query13": "SELECT * FROM CustomerOrders;",
        "query15": "SELECT * FROM CategorySalesOverview;",
    }
#  elif query_id == "query16":
#             # Get average order value by order ID (Function)
#             order_id = request.args.get("orderId", type=int)
#             cursor.execute("SELECT GetAverageOrderValueByOrderId(%s);", (order_id,))
#             query_result = cursor.fetchone()

    # # Handle execution of delete queries (query 17)
    # if query_id == "query17":
    #     conn = get_db_connection()
    #     if not conn:
    #         return jsonify({"error": "Database connection failed."}), 500

    #     cursor = conn.cursor(dictionary=True)
    #     try:
    #         # Delete from `ordered_items` first, then from `product_info`
    #         cursor.execute("DELETE FROM ordered_items WHERE Item_Id = 1;")
    #         cursor.execute("DELETE FROM product_info WHERE Product_Id = 1;")
    #         conn.commit()
    #         query_result = {"message": "Product with ID 1 and related order items deleted successfully."}
    #     except Error as e:
    #         error = f"Error executing query: {e}"
    #     finally:
    #         cursor.close()
    #         conn.close()

    #     return jsonify({"result": query_result if not error else {"error": error}})

    # Process user-input-based queries (12, 14, and 16)
    conn = get_db_connection()
    if not conn:
        return jsonify({"error": "Database connection failed."}), 500

    cursor = conn.cursor(dictionary=True)
    try:
        if query_id == "query12":
            # Update product price (Procedure)
            product_id = request.args.get("productId", type=int)
            new_price = request.args.get("newPrice", type=float)
            cursor.callproc('UpdateProductPrice', [product_id, new_price])
            conn.commit()
            cursor.execute("SELECT * FROM product_info;")
            query_result = cursor.fetchall()
        
        elif query_id == "query14":
            # Add new product (Procedure)
            product_name = request.args.get("productName", type=str)
            category_id = request.args.get("productCategory", type=int)
            price = request.args.get("price", type=float)
            cursor.callproc('AddNewProduct', [product_name, category_id, 1, price])
            conn.commit()
            cursor.execute("SELECT * FROM product_info;")
            query_result = cursor.fetchall()
        
        elif query_id == "query16":
            # Get average order value by order ID (Function)
            order_id = request.args.get("orderId", type=int)
            cursor.execute("SELECT GetAverageOrderValueByOrderId(%s);", (order_id,))
            query_result = cursor.fetchone()
            
        elif query_id == "query17":
            order_id = request.args.get("orderId", type=int)
            # Delete from `ordered_items` first, then from `product_info`
            cursor.execute("DELETE FROM ordered_items WHERE Item_Id = (%s);",(order_id,))
            # cursor.execute("DELETE FROM product_info WHERE Product_Id = 1;")
            conn.commit()
            query_result = f"Product with item_id {order_id} successfully deleted"

        elif query_id in queries:
            # Execute the regular query
            cursor.execute(queries[query_id])
            query_result = cursor.fetchall()
        
        else:
            error = "Invalid query selected."

    except Error as e:
        error = f"Error executing query: {e}"
    finally:
        cursor.close()
        conn.close()

    # Return the result or error as a JSON response
    if error:
        return jsonify({"error": error}), 500
    else:
        return jsonify({"result": query_result})

if __name__ == '__main__':
    app.run(debug=True, port=5002)
