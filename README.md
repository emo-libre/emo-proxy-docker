# EMO Proxy Docker Setup

This project provides a Docker-based environment for intercepting and proxying traffic from EMO devices, using DNS redirection, Nginx, mitmproxy, and the [waringer/emo](https://github.com/waringer/emo) proxy. It is designed to help analyze, debug, or modify requests between EMO and the living.ai API.

## Features
- **dnsmasq**: Redirects EMO API domains to your local machine for interception.
- **nginx**: Acts as a reverse proxy, forwarding HTTPS traffic to mitmproxy and handling SSL termination.
- **mitmproxy**: Intercepts and forwards traffic, with a web interface for inspection and modification.
- **emo proxy ([waringer/emo](https://github.com/waringer/emo))**: Analyzes and proxies EMO requests, providing detailed inspection and logging.
- **Configurable**: Easily switch between proxying to the local Go app or directly to living.ai.

## How It Works
1. **dnsmasq** intercepts DNS queries from EMO and redirects API domains to your local machine (default: `192.168.12.1`).
2. **nginx** listens on ports 80 and 443, forwarding requests to mitmproxy on port 8082.
3. **mitmproxy** receives traffic, allowing inspection and modification via its web UI (default: `http://localhost:8081`).
4. The [waringer/emo](https://github.com/waringer/emo) proxy can be used to analyze and log EMO requests. You can run it as a Docker container (see commented section in `docker-compose.yml`) or directly from your terminal for more flexibility.

## Usage
### App
Install the app made by JoVe13: https://github.com/JoVe13/emoProxy-App

### Install and run manually
### Prerequisites
- Docker and Docker Compose installed
- SSL certificates placed in `nginx/ssl/` (self-signed)

### Start the Services
```bash
docker-compose up -d
```

### Run the EMO Proxy (waringer/emo)
You can run the EMO proxy either as a Docker container (see the commented `emo-proxy` section in `docker-compose.yml`) or directly from your terminal:

```bash
git clone https://github.com/waringer/emo.git
cd emo/Proxy
go run emoProxy.go
```

### Access mitmproxy Web UI
- Open [http://localhost:8081](http://localhost:8081)
- Default password: `emo`

### Customizing DNS Redirection
- Edit `dnsmasq.conf` to change which domains are redirected.

### Stopping the Services
```bash
docker-compose down
```

## Example Linux Setup (Hotspot)

If you are running this on a Linux laptop with a WiFi chip that supports hotspot mode, you can set up your system to intercept EMO traffic as follows:

1. **Clear the NAT table:**
    ```bash
    sudo iptables -t nat -F
    ```

2. **Redirect traffic from the hotspot interface (`ap0`) to local port 443:**
    ```bash
    sudo iptables -t nat -A PREROUTING -i ap0 -p tcp --dport 443 -j REDIRECT --to-port 443
    ```

3. **Allow local routing and disable reverse path filtering:**
    ```bash
    sudo sysctl -w net.ipv4.conf.all.route_localnet=1
    sudo sysctl -w net.ipv4.conf.ap0.rp_filter=0
    ```

Make sure your EMO device connects to the WiFi hotspot provided by your laptop, and that your laptop's IP matches the one set in `dnsmasq.conf` (default: `192.168.12.1`).

## Creating SSL Certificate for Nginx

To generate a self-signed SSL certificate for Nginx (required for HTTPS interception):

```bash
mkdir -p nginx/ssl
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout nginx/ssl/server.key \
  -out nginx/ssl/server.crt \
  -subj "/CN=api.living.ai"
```

Place the generated `server.crt` and `server.key` files in the `nginx/ssl/` directory.

## Notes
- All services use `network_mode: host` for direct access to local ports.
- Make sure your EMO device is configured to use your machine as its DNS server.
- For HTTPS interception, EMO must trust your Nginx SSL certificate.
- The EMO proxy ([waringer/emo](https://github.com/waringer/emo)) can be run outside Docker for easier development and debugging.

## Troubleshooting
- Check logs in Docker containers for errors.
- Use mitmproxy's web UI to inspect and debug traffic.
- Ensure firewall rules allow traffic on required ports.

## License
MIT
