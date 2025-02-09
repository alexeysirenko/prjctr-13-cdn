# Custom CDN

This is an attempt to build a test CDN network, sets up a docker cluster with a custom DNS, which redirects
you to a zone-specific load balancer based on your IP address. To simulate requests from different IPs
two separate client containers are set up, "world" and "America".

## Start the cluster

```bash
docker compose up -d
```

## DNS-based Load Balancing

- The docker cluster is configured to use Bind service as DNS by the client instances (world and american)
- Clients in `172.30.1.0/24` are considered **American** and get redirected to **172.30.0.10**.
- All other clients are redirected to the **world** load balancer.
- Note: american load balancer has caching disabled for test purposes

## Testing DNS resolution based on the client IP

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

## Testing performance with and without caching on the load balancer's level

- World load balancer uses cache for images
- American load balancer does not

```bash
docker exec -it client-world siege -t5S -f /data/urls.txt
docker exec -it client-america siege -t5S -f /data/urls.txt
```

|                     | cache (world) | no cache (America) |
| :------------------ | :-----------: | :----------------: |
| Transactions        |     2971      |        720         |
| Longest transaction |     0.13      |        0.56        |

## Testing performance with different load balancing strategies

- Using `wrk` instead of `siege` for higher concurrency
- Using only american load balancer to avoid caching
- Adjustments are made by setting a `balance` section in the Haproxy config file

```bash
docker exec -it client-america wrk -c1000 -t10 -d5s http://mypic.com/1.jpg
```

| balance          | round robin | leastconn | random |
| :--------------- | :---------: | :-------: | :----: |
| Requests         |    3125     |   3128    |  3093  |
| Latency, s (avg) |    1.26     |   1.34    |  1.30  |
| Latency, s (max) |    2.00     |   1.97    |  2.00  |
