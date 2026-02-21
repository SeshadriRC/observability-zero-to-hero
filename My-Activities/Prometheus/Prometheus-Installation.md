## 

## Step 1: Navigate to Prometheus website download folder and install it using wget


[Prometheus](https://prometheus.io/download/)

- copy link address of linux tar file
<img width="1915" height="682" alt="image" src="https://github.com/user-attachments/assets/15cc6b41-27d4-4df7-a5b9-b16b83ea3158" />

```bash
wget https://github.com/prometheus/prometheus/releases/download/v3.9.1/prometheus-3.9.1.linux-amd64.tar.gz

# Below file will get downloaded
prometheus-3.9.1.linux-amd64.tar.gz

# Unzip the tar
tar xzf prometheus-3.9.1.linux-amd64.tar.gz

root@ip-172-31-8-173:~# ls
prometheus  prometheus-3.9.1.linux-amd64  prometheus-3.9.1.linux-amd64.tar.gz  snap

# Move the  prometheus file to etc location
mv prometheus-3.9.1.linux-amd64
```

```bash
# Copy the below configuration to /etc/systemd/system/prometheus.service

vi /etc/systemd/system/prometheus.service

[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target
[Service]
ExecStart=/etc/prometheus/prometheus --config.file=/etc/prometheus/prometheus.yml
Restart=always
[Install]
WantedBy=multi-user.target

# Reload and start the prometheus daemon

systemctl daemon-reload
systemctl restart prometheus
systemctl enable prometheus
systemctl status prometheus

```
