```mermaid
sequenceDiagram
    participant User
    participant Browser
    participant DNS
    participant Nginx
    participant AppServer
    participant Database
    
    User->>Browser: Enter www.foobar.com
    Browser->>DNS: Name resolution
    DNS-->>Browser: Returns IP 8.8.8.8
    Browser->>Nginx: Sends HTTP request to 8.8.8.8
    Nginx->>AppServer: Routes the request
    AppServer->>CodeBase: Loads application files
    AppServer->>Database: Executes SQL query
    Database-->>AppServer: Returns data
    AppServer-->>Nginx: Sends application response
    Nginx-->>Browser: Delivers HTTP response
    Browser-->>User: Displays the website
````

# One-Server Web Infrastructure for www.foobar.com

This document describes the **one-server web infrastructure** that hosts the website accessible via `www.foobar.com`, detailing its components, roles, and potential issues.

## Infrastructure Overview
The architecture consists of:
- **Domain Name** (`foobar.com`) configured with a **WWW record** that resolves to the server's IP (`8.8.8.8`).
- **One server** that contains:
  - **Web Server** (Nginx)
  - **Application Server**
  - **Application Files** (Code base)
  - **Database** (MySQL)

## Components and Their Roles

### 1. What is a Server?
A **server** is a computer or system that provides services, resources, or data to clients (such as browsers). In this setup, the server hosts the website and responds to user requests.

### 2. What is the Role of the Domain Name?
The **domain name** (foobar.com) acts as a **human-readable address** for the website. Instead of remembering an IP address (`8.8.8.8`), users type `www.foobar.com`, and the **DNS system** translates it into the corresponding IP address.

### 3. What Type of DNS Record is WWW in www.foobar.com?
The `www` subdomain in `www.foobar.com` is typically defined using an **A record** (Address Record), mapping it to the IP address **8.8.8.8**. Alternatively, it could use a **CNAME** record if it pointed to another domain instead of a direct IP.

### 4. What is the Role of the Web Server?
The **web server (Nginx)**:
- Accepts incoming HTTP requests from users.
- Routes them to the **application server**.
- Serves static files (like CSS, JavaScript, images).
- Acts as a reverse proxy to improve performance.

### 5. What is the Role of the Application Server?
The **application server**:
- Processes dynamic requests (like fetching user data).
- Runs the business logic (e.g., PHP, Python, Node.js).
- Communicates with the **database** to fetch/store data.

### 6. What is the Role of the Database?
The **MySQL database**:
- Stores structured data (user accounts, posts, transactions).
- Executes queries requested by the application server.
- Ensures data integrity and supports indexing for efficiency.

### 7. What is the Server Using to Communicate with the User’s Computer?
The server **communicates via HTTP(S)** using:
- **TCP/IP** protocols for data transmission.
- **Nginx** handling HTTP requests and responses.
- **Browser** interpreting and rendering the received data.

## Infrastructure Issues

While this setup works, it has **major limitations**:

### 1. **Single Point of Failure (SPOF)**
Since **everything is hosted on one server**, if that server crashes:
- The website becomes **inaccessible**.
- No redundancy exists to handle failures.

### 2. **Downtime During Maintenance**
- If **deploying new code**, the **web server may need a restart**, causing downtime.
- No backup server exists to **serve requests** while updates occur.

### 3. **Limited Scalability**
- A single server **cannot handle excessive traffic**.
- High demand may result in **slow responses or crashes**.
- No **load balancer** or distributed architecture is in place.

## Conclusion
This setup is simple and functional but **not ideal** for high-traffic or mission-critical applications. A **multi-server infrastructure**, **load balancing**, and **replication** would improve reliability and scalability.

