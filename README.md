# API Gateway

## Overview
This API Gateway is a centralized entry point for managing and routing API requests to multiple backend microservices. It provides authentication, rate limiting, caching, load balancing, and logging to ensure scalable and secure communication between clients and backend services.

## Features
- **JWT Authentication**: Secures API endpoints with token-based authentication.
- **Rate Limiting**: Prevents API abuse by limiting requests per IP.
- **Caching (Redis)**: Caches API responses to reduce backend load and improve response times.
- **Load Balancing (HAProxy)**: Distributes incoming traffic across multiple API Gateway instances.
- **Reverse Proxy (Nginx)**: Handles SSL termination and request forwarding.
- **Logging & Monitoring**: Uses Morgan and Prometheus for tracking API performance.
- **Service Discovery**: Dynamically routes traffic to available backend services.

---

## Tech Stack
- **Backend**: Node.js (Express.js)
- **Authentication**: JSON Web Tokens (JWT)
- **Caching**: Redis
- **Load Balancing**: HAProxy
- **Reverse Proxy & SSL Termination**: Nginx
- **Logging & Monitoring**: Morgan, Prometheus, Grafana
- **Database (for logging API usage)**: PostgreSQL/MongoDB

---

## System Architecture

```
(Client) --> (Nginx: SSL, Caching, Rate Limiting) --> (HAProxy: Load Balancing) --> (API Gateway) --> (Backend Services)
```

- **Nginx**: Handles SSL termination, request caching, and filtering.
- **HAProxy**: Load balances traffic across API Gateway instances.
- **API Gateway**: Routes requests to backend services, performs authentication, and rate limiting.
- **Backend Services**: The actual microservices handling business logic.

---

## Installation & Setup

### **1. Clone the Repository**
```bash
git clone https://github.com/your-repo/api-gateway.git
cd api-gateway
```

### **2. Install Dependencies**
```bash
npm install
```

### **3. Set Up Environment Variables**
Create a `.env` file in the root directory:
```env
PORT=3000
JWT_SECRET=your_jwt_secret
REDIS_HOST=127.0.0.1
REDIS_PORT=6379
SERVICE_USER=http://localhost:5001
SERVICE_ORDER=http://localhost:5002
SERVICE_PRODUCT=http://localhost:5003
```

### **4. Start Redis Server**
```bash
redis-server
```

### **5. Start API Gateway**
```bash
node server.js
```

---

## Usage
### **1. Authenticating Requests**
Send a POST request to authenticate and receive a JWT token:
```bash
curl -X POST http://localhost:3000/auth/login -d '{"username":"test", "password":"password"}'
```
### **2. Calling a Backend Service via API Gateway**
Make an authenticated request to a backend service:
```bash
curl -X GET http://localhost:3000/api/user/profile -H "Authorization: Bearer <TOKEN>"
```

### **3. Rate Limiting**
The API Gateway limits each IP to **100 requests per minute**. Exceeding this limit results in:
```json
{
  "message": "Too many requests, please try again later."
}
```

### **4. Caching API Responses**
Requests are cached in Redis for **60 seconds** to optimize performance.

---

## **Deployment**
### **1. Setting Up HAProxy**
Install HAProxy and configure `/etc/haproxy/haproxy.cfg`:
```ini
frontend api_gateway
    bind *:80
    default_backend backend_servers

backend backend_servers
    balance roundrobin
    server gateway1 127.0.0.1:3000 check
    server gateway2 127.0.0.1:3001 check
```
Restart HAProxy:
```bash
sudo systemctl restart haproxy
```

### **2. Setting Up Nginx**
Install and configure Nginx:
```nginx
server {
    listen 443 ssl;
    server_name api.yourdomain.com;

    location / {
        proxy_pass http://127.0.0.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    ssl_certificate /etc/nginx/ssl/fullchain.pem;
    ssl_certificate_key /etc/nginx/ssl/privkey.pem;
}
```
Restart Nginx:
```bash
sudo systemctl restart nginx
```

---

## **Scaling & Monitoring**
### **1. Run API Gateway on Multiple Instances**
```bash
pm run start --port 3001
```

### **2. Monitor Performance Using Prometheus & Grafana**
- Install Prometheus & Grafana
- Set up metrics endpoint `/metrics`

---

## **Contributing**
We welcome contributions! Please open an issue or submit a pull request.

---

## **License**
This project is licensed under the MIT License.

