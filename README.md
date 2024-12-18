# v2ray-server-setup
This tutorial will guide you step-by-step to set up a V2Ray server using Docker Compose and connect to it from a Linux client. ((20241218150535-65nkmbh "*"))

---

## **1. Set Up the V2Ray Server**

### **Step 1.1: Install Docker and Docker Compose**

Install Docker and Docker Compose on your server.

#### On Ubuntu/Debian:

```bash
sudo apt updatesudo apt install -y docker.io docker-compose
```

Verify the installations:

```bash
docker --versiondocker-compose --version
```

---

### **Step 1.2: Create a Project Directory**

Create a directory for your V2Ray setup:

```bash
mkdir -p ~/v2ray-servercd ~/v2ray-server
```

---

### **Step 1.3: Create the V2Ray Configuration File**

Create a `config.json` file inside the directory:

```bash
nano config.json
```

Paste the following content:

```json
{  "log": {    "access": "/var/log/v2ray/access.log",    "error": "/var/log/v2ray/error.log",    "loglevel": "info"  },  "inbounds": [    {      "port": 12345,      "protocol": "vmess",      "settings": {        "clients": [          {            "id": "replace-with-your-uuid",            "alterId": 64,            "email": "client1@example.com"          }        ]      },      "streamSettings": {        "network": "tcp"      }    }  ],  "outbounds": [    {      "protocol": "freedom",      "settings": {}    }  ]}
```

#### Replace `replace-with-your-uuid`:

Generate a UUID using Python:

```bash
python3 -c "import uuid; print(uuid.uuid4())"
```

Copy the generated UUID and replace `replace-with-your-uuid`.

---

### **Step 1.4: Create the Docker Compose File**

Create a `docker-compose.yml` file in the same directory:

```bash
nano docker-compose.yml
```

Paste the following content:

```yaml
version: '3'services:  v2ray:    image: v2fly/v2fly-core    container_name: v2ray    restart: unless-stopped    volumes:      - ./config.json:/etc/v2ray/config.json      - ./logs:/var/log/v2ray    ports:      - "12345:12345"    command: ["run", "-c", "/etc/v2ray/config.json"]
```

---

### **Step 1.5: Start the V2Ray Server**

Run the following command to start the V2Ray server:

```bash
docker-compose up -d
```

Verify the service is running:

```bash
docker-compose ps
```

---

### **Step 1.6: Check Logs**

View the V2Ray logs to ensure everything is running correctly:

```bash
docker-compose logs v2ray
```

---

### **Step 1.7: Allow Traffic Through Firewall**

If you have a firewall enabled (e.g., UFW), allow traffic on port `12345`:

```bash
sudo ufw allow 12345/tcp
```

---

## **2. Configure the Linux Client**

### **Step 2.1: Install V2Ray**

On your Linux client, install V2Ray. You can use the official script to install it:

```bash
bash <(curl -L https://raw.githubusercontent.com/v2fly/fhs-install-v2ray/master/install-release.sh)
```

---

### **Step 2.2: Create the Client Configuration File**

Create a client configuration file at `/usr/local/etc/v2ray/config.json`:

```bash
sudo nano /usr/local/etc/v2ray/config.json
```

Paste the following content:

```json
{  "log": {    "loglevel": "info"  },  "inbounds": [    {      "port": 1080,      "protocol": "socks",      "settings": {        "udp": true,        "auth": "noauth"      }    }  ],  "outbounds": [    {      "protocol": "vmess",      "settings": {        "vnext": [          {            "address": "your-server-ip",            "port": 12345,            "users": [              {                "id": "replace-with-your-uuid",                "alterId": 64,                "security": "auto"              }            ]          }        ]      }    }  ]}
```

#### Replace:

* `your-server-ip`: The IP address or domain name of your V2Ray server.
* `replace-with-your-uuid`: The same UUID you generated earlier.

---

### **Step 2.3: Start the V2Ray Client**

Start the V2Ray client with the following command:

```bash
sudo systemctl start v2ray
```

Enable it to start on boot:

```bash
sudo systemctl enable v2ray
```

---

### **Step 2.4: Verify the Connection**

Check the client logs to ensure the connection is successful:

```bash
journalctl -u v2ray -f
```

---

### **Step 2.5: Configure Proxy Settings on Linux**

* Set the system proxy settings to use `localhost:1080` as a SOCKS5 proxy.
* Alternatively, use tools like `proxychains` or a browser extension (e.g., FoxyProxy) to route traffic through the V2Ray client.

---

## **3. Optional Enhancements**

### **Step 3.1: Use TLS and WebSocket**

For better security and obfuscation, configure V2Ray to use **TLS** with **WebSocket**. Let me know if you need instructions for this step!

### **Step 3.2: Monitor with Logs**

Monitor server traffic:

```bash
docker-compose logs -f v2ray
```

---

You're all set! Let me know if you face any issues or want additional features, like configuring multiple clients or enabling advanced routing.
