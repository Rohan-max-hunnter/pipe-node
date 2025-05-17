# Pipe tesnet-node 

# 1. System Requirements
Recommended specifications:
- 4+ CPU cores

- 16GB+ RAM

- SSD storage with 100GB+ available space

- 1Gbps+ network connection

Let's start to setup pipe tesnet node

# 2. Preparing Your System
Create a dedicated user (optional but recommended)
```
sudo su -
```
```
sudo useradd -m -s /bin/bash popcache
```
```
sudo usermod -aG sudo popcache
```
Install required dependencies
```
sudo apt update
sudo apt install -y libssl-dev ca-certificates
```
Optimize Network Performance
Create a sysctl configuration file to enhance TCP performance:
```
sudo bash -c 'cat > /etc/sysctl.d/99-popcache.conf << EOL
net.ipv4.ip_local_port_range = 1024 65535
net.core.somaxconn = 65535
net.ipv4.tcp_low_latency = 1
net.ipv4.tcp_fastopen = 3
net.ipv4.tcp_slow_start_after_idle = 0
net.ipv4.tcp_window_scaling = 1
net.ipv4.tcp_wmem = 4096 65536 16777216
net.ipv4.tcp_rmem = 4096 87380 16777216
net.core.wmem_max = 16777216
net.core.rmem_max = 16777216
EOL'
```
To make sure, run this
```
sudo sysctl -p /etc/sysctl.d/99-popcache.conf
```
Increase File Limits
Set file limits for the popcache user:
```
  sudo bash -c 'cat > /etc/security/limits.d/popcache.conf << EOL
*    hard nofile 65535
*    soft nofile 65535
EOL'
```
Note:- You may need to log out and back in for these changes to take effect.

# 3. Installation
 Create Installation Directory
 ```
sudo mkdir -p /opt/popcache
cd /opt/popcache
```
 Download and Install Binaries
Note: The specific download links for the binaries are provided after registration. Once you have them, download and set executable permissions:
```
wget https://download.pipe.network/static/pop-v0.3.0-linux-x64.tar.gz
tar -xzf pop-v0.3.0-linux-x64.tar.gz
sudo mv pop /usr/local/bin/
sudo chmod +x /usr/local/bin/pop
pop --help
```
# 4. Configuration
Create and edit the (config.json) file:
```
nano config.json
```
Insert the following configuration, adjusting values as needed:
```
{
  "pop_name": "your-pop-name",
  "pop_location": "Your Location, Country",
  "invite_code": "your-invite-code",
  "server": {
    "host": "0.0.0.0",
    "port": 443,
    "http_port": 80,
    "workers": 40
  },
  "cache_config": {
    "memory_cache_size_mb": 4096,
    "disk_cache_path": "./cache",
    "disk_cache_size_gb": 100,
    "default_ttl_seconds": 86400,
    "respect_origin_headers": true,
    "max_cacheable_size_mb": 1024
  },
  "api_endpoints": {
    "base_url": "https://dataplane.pipenetwork.com"
  },
  "identity_config": {
    "node_name": "your-node-name",
    "name": "Your Name",
    "email": "your.email@example.com",
    "website": "https://your-website.com",
    "twitter": "your_twitter_handle",
    "discord": "your_discord_username",
    "telegram": "your_telegram_handle",
    "solana_pubkey": "YOUR_SOLANA_WALLET_ADDRESS_FOR_REWARDS"
  }
}
```
Configuration tips:
1. **Memory settings**:

   - Set `memory_cache_size_mb` based on your available RAM (50-70% of total RAM is a good starting point)

   - Example: For a 16GB RAM server, set to 8192-10240

2. **Disk settings**:

   - Set `disk_cache_size_gb` based on your available disk space (leave at least 20% free)

   - Example: For a 500GB disk, set to 350-400

3. **Worker settings**:

   - For best performance, set `workers` to 0 which auto-detects the number of CPU cores

   - For specific control, set to the number of CPU cores minus 1

4. **Network binding**:

   - Set `server.host` to `0.0.0.0` to bind to all network interfaces

   - For security in development, use `127.0.0.1` to only accept local connections

5. **Identity configuration**:

   - All fields under `identity_config` are important for proper attribution

   - The `solana_pubkey` field is required for receiving rewards
     
6. **For VPS user put the country name that your VPS provider provides you.**

![image](https://github.com/user-attachments/assets/3394e312-137e-4734-8029-97c407f382a8)

After putting all information or values use
`Ctrl+X` then `Y` press `Enter`

# 5. Creating a Systemd Service
 Create a systemd service file for automatic startup and management:

To edit the popcache service configuration using the nano text editor, run the following command:
```
sudo nano /etc/systemd/system/popcache.service
```
Put this 
```
[Unit]

Description=POP Cache Node

After=network.target

[Service]

Type=simple

User=root

Group=root

WorkingDirectory=/opt/popcache

ExecStart=/opt/popcache/pop

Restart=always

RestartSec=5

LimitNOFILE=65535

StandardOutput=append:/opt/popcache/logs/stdout.log

StandardError=append:/opt/popcache/logs/stderr.log

Environment=POP_CONFIG_PATH=/opt/popcache/config.json

[Install]

WantedBy=multi-user.target
```
Log rotation
```
sudo nano /etc/logrotate.d/popcache
```
add this ' ` ' 3 times like shown in image 

![image](https://github.com/user-attachments/assets/8587dc8b-9cbe-46d7-aeab-a8310dbc7e22)

```

/opt/popcache/logs/*.log {

    daily

    missingok

    rotate 14

    compress

    delaycompress

    notifempty

    create 0640 popcache popcache

    sharedscripts

    postrotate

        systemctl reload popcache >/dev/null 2>&1 || true

    endscript

}

```
# 6. Firewall Configuration
only if you enable ufw in your vps or skip this
```
sudo ufw allow 443/tcp
sudo ufw allow 80/tcp
```
# Then start Pipe node
```
sudo systemctl enable popcache
```
```
sudo systemctl start popcache
```
```
sudo systemctl daemon-reload
```
 # For VPS user 
 ```
sudo apt update && apt install screen -y
```
```
screen -S pipe
```
```
cd /opt/popcache
```
```
./pop --validate-config
```
# 💻Next day command
```
cd /opt/popcache
```
```
sudo systemctl enable popcache
```
```
sudo systemctl start popcache
```
```
sudo systemctl daemon-reload
```
```
./pop --validate-config
```

# - Don't forget to join TG channel!
- https://t.me/realdrophunter
