```mermaid
sequenceDiagram
    participant User
    participant Browser
    participant DNS
    participant HAProxy
    participant Server1
    participant Server2
    participant Nginx1
    participant Nginx2
    participant AppServer1
    participant AppServer2
    participant Database1
    participant Database2
    
    User->>Browser: Enter www.foobar.com
    Browser->>DNS: Name resolution request
    DNS-->>Browser: Returns IP of Load Balancer
    Browser->>HAProxy: Sends HTTP request
    HAProxy->>Server1: Routes request (or Server2)
    
    Server1->>Nginx1: Handles request (or Server2->>Nginx2)
    Nginx1->>AppServer1: Sends request to application server (or Nginx2->>AppServer2)
    
    AppServer1->>Database1: Queries database (or AppServer2->>Database2)
    Database1-->>AppServer1: Returns data (or Database2-->>AppServer2)
    
    AppServer1-->>Nginx1: Sends processed
````
# Three-Server Web Infrastructure for www.foobar.com

## **Infrastructure Overview**
This setup includes:
- **HAProxy Load Balancer** → Distributes incoming requests efficiently.
- **Two Backend Servers** → Each contains:
  - **Web Server (Nginx)** → Handles HTTP requests.
  - **Application Server** → Processes business logic.
  - **Application Files** → The code base.
  - **Database (MySQL, Primary-Replica setup)** → Stores data.

---

## **Why Each Element Is Added**
1. **Load Balancer (HAProxy)**
   - Purpose: Distributes traffic between backend servers.
   - Benefit: Prevents overloading a single server, improving availability and performance.

2. **Multiple Backend Servers**
   - Purpose: Reduces single points of failure (SPOF).
   - Benefit: Improves scalability—if one server fails, the other can still serve requests.

3. **Database Replication (Primary-Replica)**
   - Purpose: Enhances data redundancy and distributes query loads.
   - Benefit: Improves read performance and failover handling in case of primary database failure.

---

## **Load Balancer Distribution Algorithm**
**HAProxy can use different algorithms to distribute traffic. Common ones include:**
- **Round Robin (default)** → Sends requests to servers in sequential order.
- **Least Connections** → Directs traffic to the server handling the fewest active connections.
- **Source Hashing** → Ensures the same client always reaches the same backend server.

**Recommended Algorithm for This Setup:**  
**Round Robin** → Evenly distributes traffic across both backend servers for balanced load handling.

---

## **Active-Active vs. Active-Passive Setup**
### **Active-Active Load Balancing**
**Both backend servers process requests simultaneously**.  
**Advantages:**
- Maximizes resource utilization.
- Improves scalability for high traffic.

**Risk:**  
- Requires **session persistence handling**, or users may switch between servers mid-session.

---

### **Active-Passive Load Balancing**
**One server handles requests, the other is on standby**.  
**Advantages:**
- Provides **failover support**—if the active server fails, traffic moves to the passive one.

**Risk:**  
- Only one server is utilized at a time, reducing efficiency.

**Recommended Setup for This Infrastructure:**  
**Active-Active** → To ensure **efficient load distribution and high availability**.

---

## **How Primary-Replica (Master-Slave) Database Cluster Works**
### **Database Replication Overview**
- **Primary (Master) Node** → Handles **writes** and distributes updates to replicas.
- **Replica (Slave) Nodes** → Handle **reads**, offloading query loads.

### **How It Benefits the System**
- **Increases availability** → If the primary fails, a replica can take over.
- **Improves read performance** → Queries are spread across multiple database nodes.

---

## **Difference Between Primary Node and Replica Node**
| Feature            | Primary Node (Master) | Replica Node (Slave) |
|--------------------|----------------------|----------------------|
| Handles Write Ops | Yes | No |
| Handles Read Ops  | Yes | Yes |
| Syncs Data to Others | Yes | No |
| Used by the Application | For writes | For reads |

🛠️ **In application logic**:  
- **Writes (e.g., user registration)** → Go to **Primary**.  
- **Reads (e.g., product listings)** → Go to **Replica**.

---

## **Infrastructure Issues**
### **1. Single Points of Failure (SPOF)**
**Critical SPOF areas:**
- **HAProxy Load Balancer** → If it fails, no traffic reaches backend servers.
- **Primary Database Node** → If the primary goes down without proper failover, **writes** are lost.
- **Each Backend Server** → If a server fails and isn’t replaced, **half of capacity** is lost.

**Mitigation Strategies:**
- Redundant **failover load balancer**.
- **Automatic database failover** system.
- **Auto-scaling backend servers**.

---

### **2. Security Issues**
Major weaknesses:
- **No Firewall** → The infrastructure is **directly exposed**, increasing risks.
- **No HTTPS Encryption** → User data is **unencrypted**, vulnerable to attacks.

**Fixes:**
- **Firewall Setup (UFW, iptables)** → Blocks unauthorized traffic.
- **Enable HTTPS (Let’s Encrypt)** → Encrypts requests for security.

---

### **3. No Monitoring**
Without monitoring, failures go unnoticed:
- **No alerts** for high CPU usage, downtime, or failed requests.
- **No tracking** for database performance or replication lag.

**Fixes:**
- **Use Prometheus + Grafana** → Real-time monitoring.
- **Set up alerting** for failures and resource usage.

---

## **Final Thoughts**
This infrastructure **boosts availability and load balancing**, but lacks security, monitoring, and failsafe mechanisms.
Strengthening **fault tolerance, security, and proactive monitoring** will make it **production-ready**. 


