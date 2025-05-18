```mermaid
graph TD;
    User["User requests www.foobar.com"] -->|DNS Lookup| DNS["DNS resolves to Load Balancer"];
    DNS -->|HTTPS Request| HAProxy["HAProxy Load Balancer (HTTPS Enabled)"];
    
    HAProxy -->|Routes Secure Traffic| Firewall1["Firewall 1"];
    Firewall1 -->|Filters Traffic| Server1["Backend Server 1"];
    
    HAProxy -->|Routes Secure Traffic| Firewall2["Firewall 2"];
    Firewall2 -->|Filters Traffic| Server2["Backend Server 2"];

    HAProxy -->|Routes Secure Traffic| Firewall3["Firewall 3"];
    Firewall3 -->|Filters Traffic| Server3["Monitoring & Database"];

    Server1 -->|Web Handling| Nginx1["Nginx Web Server"];
    Server2 -->|Web Handling| Nginx2["Nginx Web Server"];
    
    Nginx1 -->|Forwards to App| AppServer1["Application Server"];
    Nginx2 -->|Forwards to App| AppServer2["Application Server"];
    
    AppServer1 -->|Access Code| CodeBase1["Application Files"];
    AppServer2 -->|Access Code| CodeBase2["Application Files"];
    
    AppServer1 -->|Database Query| Database1["MySQL Primary Database"];
    AppServer2 -->|Database Query| Database2["MySQL Replica Database"];
    
    Database1 -->|Sync Data| Database2;
    
    Server3 -->|Monitoring Data Collection| Monitor1["Sumologic / Prometheus Agent"];
    Server3 -->|Monitor HAProxy, Servers| Monitor2["Monitoring Client"];
    
    SSL["SSL Certificate (HTTPS)"] --> HAProxy;
````
# **Three-Server Web Infrastructure for www.foobar.com (Secured, Encrypted, and Monitored)**

## **Infrastructure Overview**
This architecture enhances security, ensures encrypted traffic, and enables monitoring capabilities. It consists of:

- **HAProxy Load Balancer** → Distributes traffic while serving HTTPS requests.
- **Two Backend Servers** → Each hosts:
  - **Web Server (Nginx)**
  - **Application Server**
  - **Application Files**
  - **Database (Primary-Replica MySQL setup)**
- **Monitoring & Database Server** → Handles log collection and database replication.
- **Three Firewalls** → Protects each server from unauthorized access.
- **SSL Certificate** → Encrypts traffic for secure communication.
- **Monitoring Clients** → Collects metrics and logs using Sumologic or Prometheus.

---

## **Detailed Explanation of Each Element**
### **1. Why Each Additional Element Was Added**
- **Firewalls** → Protect each server by filtering inbound and outbound traffic.
- **SSL Certificate** → Enables HTTPS encryption to secure user connections.
- **Monitoring Clients** → Tracks system health and performance.

### **2. What Firewalls Are For**
- Block unauthorized access attempts.
- Prevent **DDoS attacks** or intrusion attempts.
- Restrict traffic to **only necessary services** (e.g., web ports, database access).

### **3. Why Traffic Is Served Over HTTPS**
**Data Encryption** → Ensures user data (like passwords) is protected.  
**Security Compliance** → Many modern browsers **block non-HTTPS sites**.  
**Prevents Man-in-the-Middle Attacks** → Reduces the risk of data interception.

### **4. What Monitoring Is Used For**
- **Detecting performance issues** (high CPU, memory spikes).
- **Tracking server health** to prevent failures.
- **Logging errors and events** for debugging purposes.
- **Ensuring uptime** by alerting admins in case of outages.

### **5. How the Monitoring Tool Collects Data**
- **Metrics Collection** → Pulls data from **web servers, databases, and HAProxy**.
- **Log Aggregation** → Stores logs for later analysis.
- **Alerting System** → Notifies admins when predefined thresholds are exceeded.

### **6. Monitoring Web Server QPS (Queries Per Second)**
If you want to **monitor your web server’s QPS**, you can:
1. Use **Nginx logs** to track request count.
2. Enable **HAProxy statistics** to monitor traffic distribution.
3. Configure **Prometheus or Sumologic** to collect **live metrics**.
4. Set up alerts when **QPS exceeds expected thresholds**.

---

## **Infrastructure Issues**
### **1. Why Terminating SSL at the Load Balancer Level Is an Issue**
**HAProxy terminates HTTPS** but forwards traffic **unencrypted** to backend servers.
- Data between the **load balancer and backend servers** is **not encrypted**.
- If an attacker **gains access to the internal network**, they can intercept sensitive data.

**Solution:**  
- **Use TLS passthrough** to forward HTTPS traffic directly to backend servers.
- **Configure backend servers** to handle SSL encryption.

### **2. Why Having Only One MySQL Server Accepting Writes Is an Issue**
**Database Bottleneck** → All writes go to the **Primary**, creating a **single point of failure**.  
**If the Primary Fails** → No new data can be stored, affecting website functionality.

**Solution:**  
- Implement **automatic failover** (using MySQL Group Replication or Galera Cluster).
- Use **multi-primary replication** to allow multiple servers to handle writes.

### **3. Why Having Servers with All the Same Components Might Be a Problem**
**Limited Scalability** → Each server runs a **database, web server, and app server**, meaning all need to scale simultaneously.  
**Resource Conflicts** → If one component (e.g., MySQL) consumes too much CPU, **it impacts the entire server**.  
**High Complexity** → Managing **replicated databases** on all backend servers is difficult.

**Solution:**  
- **Separate concerns** → Run **dedicated database servers** instead of including MySQL on backend machines.
- **Implement containerization (Docker, Kubernetes)** → Isolate components for better scalability.

---

## **Final Thoughts**
This infrastructure is **secure, encrypted, and monitored**, but **optimizations are needed** to eliminate **single points of failure**, improve **data security**, and simplify management.

Would you like recommendations for **auto-scaling** or **redundant HAProxy failover**?
