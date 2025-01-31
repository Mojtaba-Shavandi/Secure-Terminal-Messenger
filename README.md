### README (Standardized English Version)

# Terminal Chat Server

This repository contains the **server-side** implementation of a secure terminal-based messaging system.

## Features

- **Admin Panel**: Upon running the server, you will see the following menu for administrative controls:

  ```
  Server started on 0.0.0.0:12345

  Admin Menu:
  1. List Users
  2. Add User
  3. Change Password
  4. Delete User
  5. Show Online Users
  6. Exit Admin Menu
  Select an option:
  ```

- **Inspired by "Nikita"**: This script is inspired by the secret messenger featured in the TV series *Nikita*.
- **End-to-End Encryption**: All messages are fully encrypted.
- **Terminal-Based Connectivity**: Users can connect using `nc <server_ip> <port>`.
- **One-on-One Messaging**: This server is designed for private two-person conversations.
- **Real-Time Connection Requests**:
  - Once logged in, a user can wait for incoming chat requests.
  - Alternatively, they can enter another user's username to send a request if the recipient is online.
- **No Message Logging**:
  - All messages are deleted after the session ends.
  - There is no caching or logging of conversations, even on the server.

## How to Use

1. **Start the Server**:  
   Run the script to launch the chat server.
   
2. **User Authentication**:  
   Users must log in before they can send or receive messages.

3. **Secure Messaging**:  
   All conversations are encrypted and automatically deleted after the session.

This project prioritizes **privacy** and **security**, ensuring that no chat history is stored anywhere.
