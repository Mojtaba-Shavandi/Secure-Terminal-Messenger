import socket
import threading
import sqlite3
import json
from cryptography.fernet import Fernet
import time

# تولید کلید رمزنگاری (در محیط واقعی بهتره ذخیره بشه)
KEY = Fernet.generate_key()
cipher = Fernet(KEY)

# تنظیمات سرور
HOST = "0.0.0.0"
PORT = 12345
users = {}  # نگه داشتن اطلاعات کاربران آنلاین
pending_requests = {}  # درخواست‌های چت در حال انتظار
lock = threading.Lock()

def init_db():
    conn = sqlite3.connect("users.db")
    cursor = conn.cursor()
    cursor.execute("""
        CREATE TABLE IF NOT EXISTS users (
            username TEXT PRIMARY KEY,
            password TEXT
        )
    """)
    conn.commit()
    conn.close()

def admin_menu():
    while True:
        print("\nAdmin Menu:")
        print("1. List Users")
        print("2. Add User")
        print("3. Change Password")
        print("4. Delete User")
        print("5. Show Online Users")
        print("6. Exit Admin Menu")
        choice = input("Select an option: ")
        
        conn = sqlite3.connect("users.db")
        cursor = conn.cursor()
        
        if choice == "1":
            cursor.execute("SELECT username FROM users")
            users_list = cursor.fetchall()
            print("Users:", [user[0] for user in users_list])
        elif choice == "2":
            username = input("Enter new username: ")
            password = input("Enter password: ")
            cursor.execute("INSERT INTO users (username, password) VALUES (?, ?)", (username, password))
            conn.commit()
            print("User added successfully.")
        elif choice == "3":
            username = input("Enter username: ")
            new_password = input("Enter new password: ")
            cursor.execute("UPDATE users SET password = ? WHERE username = ?", (new_password, username))
            conn.commit()
            print("Password updated successfully.")
        elif choice == "4":
            username = input("Enter username to delete: ")
            cursor.execute("DELETE FROM users WHERE username = ?", (username,))
            conn.commit()
            print("User deleted successfully.")
        elif choice == "5":
            print("Online Users:", list(users.keys()))
        elif choice == "6":
            conn.close()
            break
        else:
            print("Invalid option. Try again.")
        conn.close()

def handle_client(client, addr):
    client.send(b"Welcome to the chat server!\n")
    
    # احراز هویت کاربر
    client.send(b"Enter username: ")
    username = client.recv(1024).decode().strip()
    client.send(b"Enter password: ")
    password = client.recv(1024).decode().strip()
    
    # بررسی کاربر در دیتابیس
    conn = sqlite3.connect("users.db")
    cursor = conn.cursor()
    cursor.execute("SELECT password FROM users WHERE username = ?", (username,))
    row = cursor.fetchone()
    conn.close()
    
    if row:
        if row[0] != password:
            client.send(b"Incorrect password!\n")
            client.close()
            return
    else:
        client.send(b"New user registered!\n")
        conn = sqlite3.connect("users.db")
        cursor = conn.cursor()
        cursor.execute("INSERT INTO users (username, password) VALUES (?, ?)", (username, password))
        conn.commit()
        conn.close()
    
    with lock:
        users[username] = client
    
    client.send(b"You are now in the chat room. Press ENTER to see available users or type a username to start chatting.\n")
    
    while True:
        client.send(b"Enter username to chat or press ENTER to wait: ")
        target_user = client.recv(1024).decode().strip()
        
        if target_user == "":
            client.send(b"Waiting for chat requests...\n")
            pending_requests[username] = client
            while username in pending_requests:
                time.sleep(1)
            target_user = pending_requests.pop(username, None)
            if target_user is None:
                continue
        
        if target_user not in users:
            client.send(b"User not available. Waiting...\n")
            while target_user not in users:
                time.sleep(1)
            client.send(b"User is now online! Sending request...\n")
        
        target_client = users[target_user]
        target_client.send(f"{username} wants to chat with you. Accept? (yes/no): ".encode())
        response = target_client.recv(1024).decode().strip().lower()
        
        if response == "yes":
            start_chat(client, target_client, username, target_user)
        else:
            client.send(b"Request denied.\n")

def start_chat(client1, client2, user1, user2):
    client1.send(f"Connected to {user2}. Type messages below:\n".encode())
    client2.send(f"Connected to {user1}. Type messages below:\n".encode())
    
    def forward_messages(sender, receiver, sender_name):
        while True:
            try:
                message = sender.recv(1024)
                if not message:
                    break
                decrypted_message = message.decode()
                receiver.send(f"{sender_name}: {decrypted_message}\n".encode())
            except:
                break
    
    thread1 = threading.Thread(target=forward_messages, args=(client1, client2, user1))
    thread2 = threading.Thread(target=forward_messages, args=(client2, client1, user2))
    thread1.start()
    thread2.start()
    thread1.join()
    thread2.join()
    
    client1.close()
    client2.close()
    with lock:
        if user1 in users:
            del users[user1]
        if user2 in users:
            del users[user2]

def start_server():
    init_db()
    server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server.bind((HOST, PORT))
    server.listen(5)
    print(f"Server started on {HOST}:{PORT}")
    
    threading.Thread(target=admin_menu, daemon=True).start()
    
    while True:
        client, addr = server.accept()
        threading.Thread(target=handle_client, args=(client, addr)).start()

if __name__ == "__main__":
    start_server()
