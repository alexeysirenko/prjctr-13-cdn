# Cluster Setup

## Start the cluster

```bash
docker compose up -d
```

## DNS-based Load Balancing

- Clients in `172.30.1.0/24` are considered **American** and get redirected to **172.30.0.10**.
- All other clients are redirected to the **world** load balancer.

## Testing

### American Client (`172.30.1.23`)

```bash
docker exec client-america dig mypic.com  # Returns IP of America LB (A record is 172.30.0.10)
docker exec client-america curl -I mypic.com/1.jpg  # Header: x-load-balancer: lb-america
```

### World Client (Other IPs)

```bash
docker exec client-world dig mypic.com  # Returns IP of World LB (A record is 172.30.0.11)
docker exec client-world curl -I mypic.com/1.jpg  # Header: x-load-balancer: lb-world
```
