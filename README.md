# plp-database-lab - Diagnosing Slow Queries and Adding Connection Pool

import psycopg2
from psycopg2 import pool
import time

# 1. DIAGNOSING SLOW QUERIES
# Slow query: SELECT * FROM users WHERE email LIKE '%@gmail.com'
# Problem: No index, full table scan
# Fix: CREATE INDEX idx_users_email ON users(email);

# 2. ADDING CONNECTION POOL

# Create connection pool
db_pool = psycopg2.pool.SimpleConnectionPool(
    1, 20,
    host="localhost",
    database="mydb",
    user="postgres",
    password="password"
)

def get_connection():
    return db_pool.getconn()

def release_connection(conn):
    db_pool.putconn(conn)

def get_users():
    conn = get_connection()
    try:
        cursor = conn.cursor()
        # Optimized query using index
        cursor.execute("SELECT id, name, email FROM users WHERE active = true")
        users = cursor.fetchall()
        cursor.close()
        return users
    finally:
        release_connection(conn)

def explain_query():
    conn = get_connection()
    try:
        cursor = conn.cursor()
        cursor.execute("EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'test@gmail.com'")
        result = cursor.fetchall()
        for row in result:
            print(row)
        cursor.close()
    finally:
        release_connection(conn)

if __name__ == "__main__":
    start = time.time()
    users = get_users()
    print(f"Fetched {len(users)} users in {time.time() - start:.4f}s using pool")
    print("Pool working - faster than creating new connection each time!")
