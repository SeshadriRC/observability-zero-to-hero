## Prometheus-Grafana Install

---
### High-Level

1. Download prometheus software
2. Install the node exporter
3. Configure the prometheus.yml to scrape the node metrics
4. Install Grafana, Set a prometheus as a datasource
5. Add second target(EC2) to a prometheus

---
### Prometheus Install

### Step 1: Navigate to Prometheus website download folder and install it using wget


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
```

```bash
# Reload and start the prometheus daemon

systemctl daemon-reload
systemctl restart prometheus
systemctl enable prometheus
systemctl status prometheus

root@ip-172-31-8-173:~# systemctl status prometheus
● prometheus.service - Prometheus
     Loaded: loaded (/etc/systemd/system/prometheus.service; enabled; preset: enabled)
     Active: active (running) since Sat 2026-02-21 06:39:31 UTC; 9s ago
   Main PID: 1459 (prometheus)
      Tasks: 5 (limit: 1121)
     Memory: 77.4M (peak: 77.5M)
```

```bash
# Access the prometheus using port 9090 and ip should be your EC2 public ip

curl -s http://checkip.amazonaws.com  - check public ip of ec2
Eg: http://15.206.84.140:9090/

```

- Able to access prometheus

<img width="1919" height="768" alt="image" src="https://github.com/user-attachments/assets/38a3bee2-fb31-49fa-ae07-7e6a27f349cc" />

<img width="1919" height="627" alt="image" src="https://github.com/user-attachments/assets/63b3be47-c800-48d3-a1db-2f139af78db3" />


---

### Step 2: Install the node exporter

  - Node Exporter = Agent installed on server to expose system metrics to Prometheus

- copy link of linux and download using wget

<img width="1919" height="539" alt="image" src="https://github.com/user-attachments/assets/7ba29fdd-bb87-4823-94a2-b3c50656cd2a" />

```bash

root@ip-172-31-8-173:~# wget https://github.com/prometheus/node_exporter/releases/download/v1.10.2/node_exporter-1.10.2.linux-amd64.tar.gz


root@ip-172-31-8-173:~# ls
node_exporter-1.10.2.linux-amd64.tar.gz

# untar it
tar xzf node_exporter-1.10.2.linux-amd64.tar.gz

root@ip-172-31-8-173:~# ls
node_exporter-1.10.2.linux-amd64  node_exporter-1.10.2.linux-amd64.tar.gz

# Move the node exporter
mv node_exporter-1.10.2.linux-amd64 /etc/node_exporter
```
```bash
# Copy the below configuration of to /etc/systemd/system/node_exporter.service
vi /etc/systemd/system/node_exporter.service

[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target
[Service]
ExecStart=/etc/node_exporter/node_exporter
Restart=always
[Install]
WantedBy=multi-user.target
```
```bash
# Reload and start the node_exporter daemon

systemctl daemon-reload
systemctl restart node_exporter
systemctl enable node_exporter
systemctl status node_exporter

root@ip-172-31-8-173:~# systemctl status node_exporter
● node_exporter.service - Node Exporter
     Loaded: loaded (/etc/systemd/system/node_exporter.service; enabled; preset: enabled)
     Active: active (running) since Sat 2026-02-21 07:11:26 UTC; 13s ago
   Main PID: 1657 (node_exporter)
      Tasks: 3 (limit: 1121)
     Memory: 2.0M (peak: 2.3M)


curl http://localhost:9100/metrics | head    # You should see metrics output

```

---

### Step 3: Add New Target in Prometheus Config



```bash
# Just take backup for safety
cp /etc/prometheus/prometheus.yml /etc/prometheus/prometheus.yml-bkp
> /etc/prometheus/prometheus.yml
vi /etc/prometheus/prometheus.yml

global:
  scrape_interval: 15s

scrape_configs:
- job_name: node
  static_configs:
  - targets: ['172.31.8.173:9100']                  # ip address of ec2, take private ip



# After saving the file, validate the syntax by using below command
/etc/prometheus/promtool check config /etc/prometheus/prometheus.yml

root@ip-172-31-8-173:~# /etc/prometheus/promtool check config /etc/prometheus/prometheus.yml
Checking /etc/prometheus/prometheus.yml
  FAILED: parsing YAML file /etc/prometheus/prometheus.yml: yaml: unmarshal errors:
  line 2: field scrape_interval not found in type config.plain

```

```bash
# Restart the prometheus service

systemctl restart prometheus
systemctl status prometheus

root@ip-172-31-8-173:~# systemctl status prometheus
● prometheus.service - Prometheus
     Loaded: loaded (/etc/systemd/system/prometheus.service; enabled; preset: enabled)
     Active: active (running) since Sat 2026-02-21 07:23:56 UTC; 8s ago

```

- We can see below host got exposed

<img width="1919" height="631" alt="image" src="https://github.com/user-attachments/assets/0d20f110-168e-45cb-b545-caa19e7f4060" />


---

### Grafana Install

- Install grafana from apt repository- [Link](https://grafana.com/docs/grafana/latest/setup-grafana/installation/debian/#install-from-apt-repository)

```bash
sudo apt-get install -y apt-transport-https wget gnupg

sudo mkdir -p /etc/apt/keyrings/
wget -q -O - https://apt.grafana.com/gpg.key | gpg --dearmor | sudo tee /etc/apt/keyrings/grafana.gpg > /dev/null

echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list

echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com beta main" | sudo tee -a /etc/apt/sources.list.d/grafana.list

sudo apt-get update

# Installs the latest OSS release: ( Open Source Software )
sudo apt-get install grafana

```

- Restart the grafana server

```bash
systemctl restart grafana-server
systemctl enable grafana-server
systemctl status grafana-server

● grafana-server.service - Grafana instance
     Loaded: loaded (/usr/lib/systemd/system/grafana-server.service; disabled; preset: enabled)
     Active: active (running) since Sat 2026-02-21 07:37:11 UTC; 6s ago
```

- Now we can access the grafana in Port: 3000, make sure you are adding inbound rule for this port in SG

```bash
username: admin
password: admin
```

- Set a prometheus as a datasource

<img width="1919" height="620" alt="image" src="https://github.com/user-attachments/assets/99b9f82f-5d4a-48c0-ac2e-8fad5c269ece" />

- Enter the localhost and click enter

<img width="1919" height="877" alt="image" src="https://github.com/user-attachments/assets/f3bdc8bd-be18-4fc1-816c-2e91dfb23d78" />

- Save and test, it should show sucess

<img width="1546" height="360" alt="image" src="https://github.com/user-attachments/assets/1e28603a-696b-4173-9412-753f5a6cd17f" />

- Dashboard --> Import dashboard

<img width="1919" height="812" alt="image" src="https://github.com/user-attachments/assets/ee961e9b-0080-4ab3-afef-ec7b2b1cbddd" />

- Enter code 1860 and click load

<img width="1505" height="772" alt="image" src="https://github.com/user-attachments/assets/56a6cfb7-e540-4562-bfec-9d2cbd644061" />

- Click import

<img width="1918" height="847" alt="image" src="https://github.com/user-attachments/assets/08f02d9d-a6ba-40ad-8f53-f614a529786e" />

- Grafana Dashboard is ready

<img width="1919" height="930" alt="image" src="https://github.com/user-attachments/assets/6cbe6fd4-d689-4d90-9cbb-9ceaad23c212" />

---

5. Add second target(EC2) to a prometheus

- Install node_exporter - follow step 2
- Add New Target in Prometheus Config - follow step 2, below is the reference yml file

```bash
root@ip-172-31-8-173:~# cat /etc/prometheus/prometheus.yml

global:
  scrape_interval: 15s

scrape_configs:
- job_name: node
  static_configs:
  - targets: ['172.31.8.173:9100']

- job_name: second-ec2
  static_configs:
  - targets: ['172.31.9.162:9100']
```

- Now we can see the newly added targets are visible both in grafana and prometheus

<img width="1919" height="597" alt="image" src="https://github.com/user-attachments/assets/003a15d0-ad19-4170-a749-ceeb94e8ec60" />

<img width="1916" height="931" alt="image" src="https://github.com/user-attachments/assets/efd5c72e-5a34-450c-ae37-875a1e5d0866" />
