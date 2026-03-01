# Reverse Proxy

A reverse proxy sits between clients and servers, forwarding client requests to appropriate servers and returning responses.

---

## What is a Reverse Proxy?

```
Forward Proxy (client-side):
Client → Proxy → Internet → Server
         (client's identity hidden)

Reverse Proxy (server-side):
Client → Internet → Proxy → Server
                   (server's identity hidden)
```

---

## Key Functions

### 1. Load Balancing
```
Client → Reverse Proxy → Server 1
                      → Server 2
                      → Server 3
```

### 2. SSL/TLS Termination
```
Client ══HTTPS══> Proxy ──HTTP──> Server
                  (decrypts here)
```
Offloads encryption overhead from application servers.

### 3. Caching
```
Client → Proxy → Cache (hit!) → Response
               → Server (miss) → Cache → Response
```

### 4. Compression
```
Server Response: 1MB
Proxy compresses: 200KB (gzip)
Sent to Client: 200KB
```

### 5. Security
- Hide server IPs
- Web Application Firewall (WAF)
- DDoS protection
- IP blocking

---

## Nginx Configuration

```nginx
# Basic reverse proxy
server {
    listen 80;
    server_name example.com;

    location / {
        proxy_pass http://backend_servers;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}

# Load balancing
upstream backend_servers {
    least_conn;
    server 10.0.1.1:8080 weight=3;
    server 10.0.1.2:8080 weight=2;
    server 10.0.1.3:8080 weight=1;
}

# SSL termination
server {
    listen 443 ssl;
    ssl_certificate /etc/ssl/cert.pem;
    ssl_certificate_key /etc/ssl/key.pem;

    location / {
        proxy_pass http://backend_servers;
    }
}

# Caching
proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=my_cache:10m max_size=1g;

server {
    location /static/ {
        proxy_cache my_cache;
        proxy_cache_valid 200 1d;
        proxy_pass http://backend_servers;
    }
}

# Compression
gzip on;
gzip_types text/plain application/json application/javascript;
gzip_min_length 1000;
```

---

## Reverse Proxy vs Load Balancer

| Feature | Reverse Proxy | Load Balancer |
|---------|---------------|---------------|
| Primary function | Request forwarding | Traffic distribution |
| SSL termination | Yes | Sometimes |
| Caching | Yes | No |
| Compression | Yes | No |
| Content inspection | Yes (L7) | Depends (L4/L7) |
| Web serving | Yes | No |

In practice, reverse proxies often include load balancing.

---

## Common Use Cases

### 1. Microservices Gateway
```nginx
location /api/users {
    proxy_pass http://user-service;
}

location /api/orders {
    proxy_pass http://order-service;
}

location /api/products {
    proxy_pass http://product-service;
}
```

### 2. A/B Testing
```nginx
split_clients "${remote_addr}" $variant {
    50% "A";
    50% "B";
}

location / {
    if ($variant = "A") {
        proxy_pass http://version_a;
    }
    proxy_pass http://version_b;
}
```

### 3. Blue-Green Deployment
```nginx
upstream blue {
    server blue-1:8080;
    server blue-2:8080;
}

upstream green {
    server green-1:8080;
    server green-2:8080;
}

# Switch by changing this
upstream active {
    server blue-1:8080;  # Currently blue
}
```

### 4. Rate Limiting
```nginx
limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;

location /api/ {
    limit_req zone=api burst=20 nodelay;
    proxy_pass http://backend;
}
```

---

## Popular Reverse Proxies

| Name | Strengths |
|------|-----------|
| Nginx | High performance, versatile |
| HAProxy | TCP/HTTP, very fast |
| Traefik | Docker/K8s native, auto-config |
| Envoy | Service mesh, observability |
| Caddy | Auto HTTPS, simple config |

---

## Interview Talking Points

1. **Purpose**: Single entry point, security, performance
2. **Functions**: SSL termination, caching, compression, LB
3. **vs Load Balancer**: Reverse proxy does more than distribute traffic
4. **vs API Gateway**: API Gateway adds business logic (auth, rate limit)
5. **Tools**: Know Nginx configuration basics
