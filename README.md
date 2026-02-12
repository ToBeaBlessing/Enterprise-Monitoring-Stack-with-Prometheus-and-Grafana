# Enterprise Monitoring Stack with Prometheus and Grafana

![Prometheus](https://img.shields.io/badge/Prometheus-E6522C?style=for-the-badge&logo=Prometheus&logoColor=white)
![Grafana](https://img.shields.io/badge/grafana-%23F46800.svg?style=for-the-badge&logo=grafana&logoColor=white)
![AWS](https://img.shields.io/badge/AWS-%23FF9900.svg?style=for-the-badge&logo=amazon-aws&logoColor=white)
![Java](https://img.shields.io/badge/java-%23ED8B00.svg?style=for-the-badge&logo=openjdk&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black)

**Production-grade monitoring infrastructure implementing Prometheus for metrics collection with Node Exporter and Blackbox Exporter, combined with Grafana for real-time visualization dashboards, monitoring both application uptime/SSL certificates and server resource utilization (CPU, RAM, disk) across distributed systems.**

## 🎯 Project Purpose

This project demonstrates comprehensive observability practices through:

- **Multi-layered monitoring** with application uptime, SSL certificate validation, and infrastructure metrics
- **Distributed monitoring architecture** with separate application and monitoring servers
- **Real-time metrics collection** using Prometheus with custom scrape configurations
- **Advanced visualization** with Grafana dashboards showing system health and performance trends
- **Blackbox monitoring** for external website availability and SSL/TLS certificate expiration tracking
- **Infrastructure metrics** with Node Exporter for CPU, RAM, disk, and network monitoring

**Ideal for demonstrating:** Site Reliability Engineer (SRE), DevOps Engineer, Platform Engineer, Infrastructure Monitoring Specialist, or Cloud Operations Engineer role competencies.

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Components](#components)
- [Prerequisites](#prerequisites)
- [Infrastructure Setup](#infrastructure-setup)
- [Application Server Setup](#application-server-setup)
- [Monitoring Server Setup](#monitoring-server-setup)
- [Grafana Configuration](#grafana-configuration)
- [Dashboard Setup](#dashboard-setup)
- [Testing](#testing)
- [Cleanup](#cleanup)
- [Skills Demonstrated](#skills-demonstrated)
- [Troubleshooting](#troubleshooting)

## 🎯 Overview

This project showcases a complete observability stack with two main monitoring strategies:

### Website & Application Monitoring
- **Uptime Monitoring:** HTTP/HTTPS endpoint availability checks
- **SSL Certificate Monitoring:** Certificate expiration tracking and validation
- **Response Time Metrics:** Latency and performance monitoring
- **Status Code Tracking:** HTTP response code validation

### Server & Infrastructure Monitoring
- **CPU Metrics:** Usage, load average, and core-level statistics
- **Memory Metrics:** RAM usage, available memory, and swap utilization
- **Disk Metrics:** Disk space, I/O operations, and filesystem health
- **Network Metrics:** Bandwidth usage, packet statistics, and connection tracking

### Key Features

| Component | Purpose | Port | Metrics |
|-----------|---------|------|---------|
| **Prometheus** | Metrics collection & storage | 9090 | Time-series database |
| **Node Exporter** | Server hardware metrics | 9100 | CPU, RAM, disk, network |
| **Blackbox Exporter** | Website uptime & SSL | 9115 | HTTP probes, SSL checks |
| **Grafana** | Visualization & dashboards | 3000 | Real-time graphs & alerts |
| **Task Master Pro** | Sample Java application | 8080 | Application to monitor |

## 🏗️ Architecture

### System Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         Internet / Users                         │
└────────────────────────┬────────────────────────────────────────┘
                         │
         ┌───────────────┴───────────────┐
         │                               │
         ▼                               ▼
┌──────────────────────┐        ┌──────────────────────┐
│  Application Server  │        │  Monitoring Server   │
│   (EC2 t2.medium)    │        │   (EC2 t2.large)     │
│   20 GB Storage      │        │   25 GB Storage      │
└──────────────────────┘        └──────────────────────┘
         │                               │
         │                               │
    ┌────┴────┐                     ┌────┴────┐
    │         │                     │         │
    ▼         ▼                     ▼         ▼
┌────────┐ ┌──────────────┐   ┌──────────┐ ┌────────┐
│Task    │ │Node Exporter │   │Prometheus│ │Grafana │
│Master  │ │   :9100      │   │  :9090   │ │ :3000  │
│Pro     │ │              │   │          │ │        │
│:8080   │ └──────────────┘   └──────────┘ └────────┘
└────────┘                          │
                                    │
                            ┌───────┴────────┐
                            │                │
                            ▼                ▼
                    ┌──────────────┐  ┌─────────────┐
                    │  Blackbox    │  │ Scrape      │
                    │  Exporter    │  │ Targets     │
                    │  :9115       │  │             │
                    └──────────────┘  └─────────────┘
```

### Monitoring Flow

```
┌───────────────────────────────────────────────────────────────┐
│                    Data Collection Flow                        │
└───────────────────────────────────────────────────────────────┘

   Application Server                    Monitoring Server
   ──────────────────                    ─────────────────
   
   ┌──────────────┐                      ┌──────────────┐
   │ Task Master  │                      │  Prometheus  │
   │   (Java)     │                      │   (Scraper)  │
   │   Port 8080  │◄─────────────────────┤              │
   └──────────────┘    HTTP Probe        │  ┌────────┐  │
                       via Blackbox      │  │Blackbox│  │
   ┌──────────────┐                      │  │Exporter│  │
   │     Node     │                      │  └────────┘  │
   │   Exporter   │◄─────────────────────┤              │
   │   Port 9100  │    Scrape /metrics   │              │
   └──────────────┘                      └──────┬───────┘
                                                │
                                                │ Query
                                                │
                                         ┌──────▼───────┐
                                         │   Grafana    │
                                         │ Port 3000    │
                                         │              │
                                         │ Dashboards:  │
                                         │ • Node Metrics│
                                         │ • Blackbox   │
                                         └──────────────┘
```

### Metrics Collection Architecture

```
┌─────────────────────────────────────────────────────────┐
│              Prometheus Scrape Configuration             │
└─────────────────────────────────────────────────────────┘

Job 1: prometheus (self-monitoring)
   Target: localhost:9090
   Metrics: Prometheus internal metrics

Job 2: node_exporter (infrastructure metrics)
   Target: 107.21.170.89:9100 (App Server IP)
   Metrics:
   • CPU: node_cpu_seconds_total
   • Memory: node_memory_MemAvailable_bytes
   • Disk: node_filesystem_avail_bytes
   • Network: node_network_receive_bytes_total

Job 3: blackbox (website monitoring)
   Module: http_2xx (check for 200 response)
   Targets:
   • http://107.21.170.89:8080 (Task Master Pro)
   • https://prometheus.io (external site)
   • https://github.com (SSL check)
   
   Metrics:
   • probe_success (1 = up, 0 = down)
   • probe_duration_seconds (response time)
   • probe_http_status_code (HTTP status)
   • probe_ssl_earliest_cert_expiry (SSL expiration)
```

## 🔧 Components

### 1. Application Server
- **Task Master Pro:** Java Spring Boot TODO application
- **Node Exporter:** Collects server metrics (CPU, RAM, disk)
- **Purpose:** Represents production application being monitored

### 2. Monitoring Server
- **Prometheus:** Time-series database for metrics storage
- **Blackbox Exporter:** HTTP/HTTPS probing for uptime
- **Grafana:** Visualization platform for dashboards

### 3. Monitoring Targets
- **Website Uptime:** HTTP 200 checks, SSL certificate validation
- **Server Metrics:** Resource utilization and health
- **Application Health:** Response time and availability

## ✅ Prerequisites

### AWS Requirements
- AWS account with EC2 launch permissions
- Ability to create security groups
- 2 EC2 instances (t2.medium + t2.large)

### Local Tools
- SSH client (MobaXterm for Windows, or native terminal)
- Web browser for accessing Grafana
- Text editor for configuration files

### Knowledge Requirements
- Basic Linux command line
- Understanding of ports and networking
- Familiarity with monitoring concepts

## 🚀 Infrastructure Setup

### Step 1: Create Security Group

**In AWS Console:**

1. Go to EC2 → Security Groups → Create Security Group
2. **Name:** `primary-sg`
3. **Description:** Monitoring infrastructure security group
4. **VPC:** Select your VPC

**Inbound Rules:**

| Type | Port Range | Source | Description |
|------|------------|--------|-------------|
| SSH | 22 | Your IP | SSH access |
| HTTP | 80 | 0.0.0.0/0 | HTTP traffic |
| HTTPS | 443 | 0.0.0.0/0 | HTTPS traffic |
| Custom TCP | 587 | 0.0.0.0/0 | SMTP (if needed) |
| Custom TCP | 3000 | 0.0.0.0/0 | Grafana |
| Custom TCP | 8080 | 0.0.0.0/0 | Application |
| Custom TCP | 9090 | 0.0.0.0/0 | Prometheus |
| Custom TCP | 9100 | 0.0.0.0/0 | Node Exporter |
| Custom TCP | 9115 | 0.0.0.0/0 | Blackbox Exporter |
| Custom TCP | 3000-11000 | 0.0.0.0/0 | Additional services |

**Outbound Rules:**
- All traffic to 0.0.0.0/0

### Step 2: Launch EC2 Instances

**Application Server:**
1. Go to EC2 → Launch Instance
2. **Name:** `application-server`
3. **AMI:** Ubuntu Server 22.04 LTS
4. **Instance Type:** t2.medium (2 vCPU, 4 GB RAM)
5. **Key pair:** Create or select existing
6. **Security Group:** `primary-sg`
7. **Storage:** 20 GB gp3
8. Launch instance

**Monitoring Server:**
1. Go to EC2 → Launch Instance
2. **Name:** `monitoring-server`
3. **AMI:** Ubuntu Server 22.04 LTS
4. **Instance Type:** t2.large (2 vCPU, 8 GB RAM)
5. **Key pair:** Same as application server
6. **Security Group:** `primary-sg`
7. **Storage:** 25 GB gp3
8. Launch instance

**Note down both public IP addresses!**

### Step 3: Connect with MobaXterm

**Open MobaXterm and create two sessions:**

**Session 1 - Application Server:**
1. Click "Session" → "SSH"
2. **Remote host:** `<APP_SERVER_PUBLIC_IP>`
3. **Username:** `ubuntu`
4. **Advanced:** Browse to your `.pem` key file
5. Click "OK"

**Session 2 - Monitoring Server:**
1. Click "Session" → "SSH"
2. **Remote host:** `<MONITORING_SERVER_PUBLIC_IP>`
3. **Username:** `ubuntu`
4. **Advanced:** Browse to your `.pem` key file
5. Click "OK"

## 📦 Application Server Setup

**In Application Server MobaXterm session:**

### Step 1: Update System

```bash
sudo apt update
```

### Step 2: Clone Application Repository

```bash
# Clone Task Master Pro (or any Java application)
git clone https://github.com/jaiswaladi246/Task-Master-Pro.git
cd Task-Master-Pro
```

### Step 3: Install Java and Maven

```bash
# Install Java 17
sudo apt install openjdk-17-jre-headless -y

# Verify Java installation
java -version

# Install Maven
sudo apt install maven -y

# Verify Maven installation
mvn -version
```

### Step 4: Build and Run Application

```bash
# Build application
mvn package

# Navigate to target directory
cd target

# Test run (foreground)
java -jar todo-app-1.0-SNAPSHOT.jar

# Access application in browser: http://<APP_SERVER_IP>:8080
# If working, stop with Ctrl+C

# Run in background
java -jar todo-app-1.0-SNAPSHOT.jar &

# Press Enter to return to prompt
```

### Step 5: Install Node Exporter

```bash
# Go to home directory
cd ~

# Download Node Exporter
wget https://github.com/prometheus/node_exporter/releases/download/v1.10.2/node_exporter-1.10.2.linux-amd64.tar.gz

# Extract
tar -xvf node_exporter-1.10.2.linux-amd64.tar.gz

# Remove tarball
rm node_exporter-1.10.2.linux-amd64.tar.gz

# Rename directory
mv node_exporter-1.10.2.linux-amd64/ node_exporter

# Navigate to directory
cd node_exporter

# Test run (foreground)
./node_exporter

# Access metrics: http://<APP_SERVER_IP>:9100
# Access metrics endpoint: http://<APP_SERVER_IP>:9100/metrics
# If working, stop with Ctrl+C

# Run in background
./node_exporter &

# Press Enter to return to prompt
```

**Verify Node Exporter:**
- Browser: `http://<APP_SERVER_IP>:9100`
- Metrics: `http://<APP_SERVER_IP>:9100/metrics`

## 🔍 Monitoring Server Setup

**In Monitoring Server MobaXterm session:**

### Step 1: Update System

```bash
sudo apt update
```

### Step 2: Install Prometheus

```bash
# Download Prometheus
wget https://github.com/prometheus/prometheus/releases/download/v3.9.1/prometheus-3.9.1.linux-amd64.tar.gz

# Extract
tar -xvf prometheus-3.9.1.linux-amd64.tar.gz

# Remove tarball
rm prometheus-3.9.1.linux-amd64.tar.gz

# Rename directory
mv prometheus-3.9.1.linux-amd64/ prometheus

# Navigate to directory
cd prometheus

# Test run (foreground)
./prometheus

# Access Prometheus: http://<MONITORING_SERVER_IP>:9090
# Check Status → Targets to see endpoints
# Stop with Ctrl+C
```

### Step 3: Configure Prometheus

**Edit prometheus.yml:**

```bash
cd ~/prometheus
vi prometheus.yml
```

**Add these job configurations:**

```yaml
# Add after the existing prometheus job

  - job_name: "node_exporter"
    static_configs:
      - targets: ["<APP_SERVER_IP>:9100"]

  - job_name: "blackbox"
    metrics_path: /probe
    params:
      module: [http_2xx]
    static_configs:
      - targets:
        - http://<APP_SERVER_IP>:8080
        - https://prometheus.io
        - https://github.com
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: <MONITORING_SERVER_IP>:9115
```

**Validate YAML syntax:**
- Copy content to [yamllint.com](https://www.yamllint.com/)
- Fix any indentation errors
- Save file: `:wq`

### Step 4: Install Blackbox Exporter

```bash
# Go to home directory
cd ~

# Download Blackbox Exporter
wget https://github.com/prometheus/blackbox_exporter/releases/download/v0.28.0/blackbox_exporter-0.28.0.linux-amd64.tar.gz

# Extract
tar -xvf blackbox_exporter-0.28.0.linux-amd64.tar.gz

# Remove tarball
rm blackbox_exporter-0.28.0.linux-amd64.tar.gz

# Rename
mv blackbox_exporter-0.28.0.linux-amd64/ blackbox_exporter

# Navigate to directory
cd blackbox_exporter

# Run in background
./blackbox_exporter &

# Press Enter
```

### Step 5: Start Prometheus

```bash
cd ~/prometheus

# Run in background
./prometheus &

# Press Enter
```

**Verify Prometheus is scraping targets:**
1. Open browser: `http://<MONITORING_SERVER_IP>:9090`
2. Go to Status → Targets
3. You should see:
   - `prometheus` (localhost:9090) - UP
   - `node_exporter` (<APP_SERVER_IP>:9100) - UP
   - `blackbox` (your monitored sites) - UP

### Step 6: Install Grafana

```bash
cd ~

# Install dependencies
sudo apt-get install -y adduser libfontconfig1 musl

# Download Grafana
wget https://dl.grafana.com/grafana-enterprise/release/12.3.2+security-01/grafana-enterprise_12.3.2+security-01_21834523998_linux_amd64.deb

# Install Grafana
sudo dpkg -i grafana-enterprise_12.3.2+security-01_21834523998_linux_amd64.deb

# Reload systemd
sudo /bin/systemctl daemon-reload

# Enable Grafana to start on boot
sudo /bin/systemctl enable grafana-server

# Start Grafana
sudo /bin/systemctl start grafana-server

# Check status
sudo /bin/systemctl status grafana-server
```

**Access Grafana:**
- URL: `http://<MONITORING_SERVER_IP>:3000`
- Default credentials: `admin` / `admin`
- Change password when prompted

## 📊 Grafana Configuration

### Step 1: Add Prometheus Data Source

1. Log in to Grafana: `http://<MONITORING_SERVER_IP>:3000`
2. Click "Connections" (left sidebar)
3. Click "Add new connection"
4. Search for "Prometheus"
5. Click "Add new data source"

**Configuration:**
- **Name:** Prometheus
- **URL:** `http://localhost:9090`
- **Access:** Server (default)
- Leave other settings as default
- Click "Save & Test"
- You should see "Data source is working"

### Step 2: Import Node Exporter Dashboard

**Find Dashboard ID:**
1. Google: "node exporter grafana dashboard"
2. Visit: https://grafana.com/grafana/dashboards/
3. Search: "Node Exporter Full"
4. Popular dashboard ID: **1860** (Node Exporter Full)

**Import Dashboard:**
1. In Grafana, click "Dashboards" (left sidebar)
2. Click "New" → "Import"
3. Enter Dashboard ID: **1860**
4. Click "Load"
5. **Select Prometheus data source:** Prometheus
6. Click "Import"

**You should now see:**
- CPU usage graphs
- Memory utilization
- Disk space
- Network traffic
- System load

### Step 3: Import Blackbox Exporter Dashboard

**Find Dashboard ID:**
1. Google: "blackbox exporter grafana dashboard"
2. Popular dashboard ID: **7587** (Prometheus Blackbox Exporter)

**Import Dashboard:**
1. Click "Dashboards" → "New" → "Import"
2. Enter Dashboard ID: **7587**
3. Click "Load"
4. **Select Prometheus data source:** Prometheus
5. Click "Import"

**You should now see:**
- Website uptime status
- HTTP response codes
- SSL certificate expiration dates
- Response time metrics
- Probe success rate

## 🧪 Testing

### Test 1: Verify All Services Running

**Application Server:**
```bash
# Check Task Master Pro
curl http://localhost:8080

# Check Node Exporter
curl http://localhost:9100/metrics
```

**Monitoring Server:**
```bash
# Check Prometheus
curl http://localhost:9090/-/healthy

# Check Blackbox Exporter
curl http://localhost:9115/metrics

# Check Grafana
curl -I http://localhost:3000
```

### Test 2: Verify Prometheus Targets

1. Open: `http://<MONITORING_SERVER_IP>:9090`
2. Go to: Status → Targets
3. All targets should show "UP" status:
   - prometheus (1/1 up)
   - node_exporter (1/1 up)
   - blackbox (3/3 up)

### Test 3: Query Metrics in Prometheus

**In Prometheus UI (`http://<MONITORING_SERVER_IP>:9090`):**

**Test Node Exporter metrics:**
```promql
# CPU usage
100 - (avg by(instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# Memory usage
(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100

# Disk usage
100 - ((node_filesystem_avail_bytes{mountpoint="/"} * 100) / node_filesystem_size_bytes{mountpoint="/"})
```

**Test Blackbox metrics:**
```promql
# Website up/down (1 = up, 0 = down)
probe_success

# HTTP response time
probe_duration_seconds

# SSL certificate expiration (in seconds)
probe_ssl_earliest_cert_expiry - time()
```

### Test 4: Verify Grafana Dashboards

**Node Exporter Dashboard:**
- CPU usage should show real-time data
- Memory graphs should update
- Disk space should be visible
- Network traffic should show activity

**Blackbox Dashboard:**
- All monitored websites should show as "UP"
- Response times should be displayed
- SSL certificates should show days until expiration

### Test 5: Simulate Downtime

**Stop Task Master Pro:**
```bash
# SSH to application server
# Find Java process
ps aux | grep java

# Kill the process
kill <PID>
```

**Check monitoring:**
1. Wait 1-2 minutes for Prometheus to scrape
2. Refresh Blackbox dashboard in Grafana
3. Task Master Pro should show as "DOWN"
4. Prometheus targets page should show it as unreachable

**Restart application:**
```bash
cd ~/Task-Master-Pro/target
java -jar todo-app-1.0-SNAPSHOT.jar &
```

## 🧹 Cleanup

### Stop All Services

**Application Server:**
```bash
# Stop Task Master Pro
pkill -f "java.*todo-app"

# Stop Node Exporter
pkill -f node_exporter
```

**Monitoring Server:**
```bash
# Stop Prometheus
pkill -f prometheus

# Stop Blackbox Exporter
pkill -f blackbox_exporter

# Stop Grafana
sudo systemctl stop grafana-server
```

### Terminate EC2 Instances

```bash
# Via AWS Console
1. Go to EC2 → Instances
2. Select both instances
3. Instance State → Terminate Instance

# Or via AWS CLI
aws ec2 terminate-instances --instance-ids <INSTANCE_IDS>
```

### Delete Security Group

```bash
# After instances are terminated
1. Go to EC2 → Security Groups
2. Select "primary-sg"
3. Actions → Delete Security Group
```

## 💼 Skills Demonstrated

This project showcases expertise in:

### Observability & Monitoring
- ✅ **Metrics Collection:** Prometheus time-series database
- ✅ **Infrastructure Monitoring:** Node Exporter for system metrics
- ✅ **Application Monitoring:** Blackbox Exporter for uptime checks
- ✅ **Visualization:** Grafana dashboards for real-time insights
- ✅ **Alerting Setup:** (extensible for alert rules)

### System Administration
- 🖥️ **Linux Administration:** Ubuntu server management
- 📦 **Service Management:** Background process handling
- 🔧 **Configuration Management:** YAML configuration files
- 📊 **Log Analysis:** Troubleshooting with system logs

### Cloud & Infrastructure
- ☁️ **AWS EC2:** Instance provisioning and management
- 🔒 **Security Groups:** Network access control
- 🌐 **Networking:** Port management and connectivity
- 💾 **Storage Management:** Disk space and filesystem monitoring

### DevOps Practices
- 📝 **Documentation:** Comprehensive setup procedures
- 🔄 **Automation:** Background service execution
- 🧪 **Testing:** Verification of monitoring systems
- 🎯 **Best Practices:** Industry-standard monitoring stack

## 🔧 Troubleshooting

### Prometheus Not Scraping Targets

**Issue:** Targets show as "DOWN" in Prometheus

**Solution:**
```bash
# Check if target is reachable
curl http://<APP_SERVER_IP>:9100/metrics

# Verify prometheus.yml configuration
cd ~/prometheus
cat prometheus.yml

# Check Prometheus logs
cd ~/prometheus
./prometheus

# Look for scrape errors
# Verify IP addresses are correct
# Ensure security group allows traffic
```

### Node Exporter Not Accessible

**Issue:** Cannot access Node Exporter on port 9100

**Solution:**
```bash
# Check if Node Exporter is running
ps aux | grep node_exporter

# Restart if not running
cd ~/node_exporter
./node_exporter &

# Verify security group allows port 9100
# Check AWS Console → EC2 → Security Groups

# Test locally first
curl http://localhost:9100/metrics
```

### Grafana Dashboard Shows No Data

**Issue:** Dashboards are empty or show "No Data"

**Solution:**
```bash
# 1. Verify Prometheus data source
# Grafana → Connections → Data Sources → Prometheus
# Click "Save & Test" - should say "Data source is working"

# 2. Check if Prometheus has data
# Go to http://<MONITORING_IP>:9090
# Execute query: up
# Should return results

# 3. Verify dashboard is using correct data source
# Dashboard Settings → Variables
# Ensure datasource variable points to Prometheus

# 4. Check time range in dashboard
# Top right corner - ensure it's set to "Last 5 minutes" or "Last 1 hour"
```

### Blackbox Exporter Shows All Targets Down

**Issue:** All websites showing as unreachable

**Solution:**
```bash
# 1. Check Blackbox Exporter is running
ps aux | grep blackbox

# 2. Test Blackbox manually
curl "http://localhost:9115/probe?target=http://<APP_SERVER_IP>:8080&module=http_2xx"

# 3. Verify prometheus.yml blackbox configuration
cd ~/prometheus
cat prometheus.yml
# Check that replacement: line has correct IP:9115

# 4. Restart Prometheus
pkill -f prometheus
cd ~/prometheus
./prometheus &
```

### Application Not Accessible on Port 8080

**Issue:** Task Master Pro returns connection refused

**Solution:**
```bash
# 1. Check if application is running
ps aux | grep java

# 2. Restart application if needed
cd ~/Task-Master-Pro/target
java -jar todo-app-1.0-SNAPSHOT.jar &

# 3. Verify port 8080 is listening
sudo netstat -tulpn | grep 8080

# 4. Check security group allows port 8080
# AWS Console → EC2 → Security Groups

# 5. Test locally first
curl http://localhost:8080
```

### SSL Certificate Monitoring Not Working

**Issue:** SSL expiration metrics not showing

**Solution:**
```bash
# 1. Ensure targets use HTTPS (not HTTP)
# In prometheus.yml:
# - targets:
#     - https://prometheus.io  (not http://)

# 2. Test probe manually
curl "http://localhost:9115/probe?target=https://prometheus.io&module=http_2xx"

# 3. Check for SSL metrics
# In Prometheus, query:
probe_ssl_earliest_cert_expiry

# 4. Verify Blackbox Exporter supports SSL
cd ~/blackbox_exporter
cat blackbox.yml
# Should have http_2xx module configured
```

### Grafana Login Issues

**Issue:** Cannot log in to Grafana

**Solution:**
```bash
# Reset admin password
sudo grafana-cli admin reset-admin-password newpassword

# Restart Grafana
sudo systemctl restart grafana-server

# Check Grafana logs
sudo journalctl -u grafana-server -f

# Verify Grafana is running
sudo systemctl status grafana-server
```

## 📚 Additional Resources

- [Prometheus Documentation](https://prometheus.io/docs/)
- [Node Exporter Guide](https://github.com/prometheus/node_exporter)
- [Blackbox Exporter Documentation](https://github.com/prometheus/blackbox_exporter)
- [Grafana Documentation](https://grafana.com/docs/)
- [PromQL Query Language](https://prometheus.io/docs/prometheus/latest/querying/basics/)
- [Grafana Dashboard Repository](https://grafana.com/grafana/dashboards/)

## 🚀 Future Enhancements

- [ ] Configure Prometheus AlertManager for notifications
- [ ] Set up email/Slack alerts for downtime
- [ ] Add Loki for log aggregation
- [ ] Implement Prometheus federation for multiple clusters
- [ ] Configure long-term storage with Thanos
- [ ] Add custom alerting rules in Prometheus
- [ ] Implement Grafana user authentication (LDAP/OAuth)
- [ ] Create custom dashboards for application-specific metrics
- [ ] Set up Prometheus service discovery for dynamic targets
- [ ] Configure TLS/SSL for Grafana and Prometheus
- [ ] Implement backup and disaster recovery for metrics
- [ ] Add container monitoring with cAdvisor
- [ ] Configure Prometheus recording rules for performance
- [ ] Set up Grafana annotations for deployment tracking

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.

---

**Author:** Blessing Omomola  
**Institution:** Morgan State University - Industrial Engineering (MS)  
**Certifications:** AWS Solutions Architect Associate, AWS Cloud Practitioner  
**GitHub:** [@ToBeaBlessing](https://github.com/ToBeaBlessing)  
**LinkedIn:** [Connect with me](https://linkedin.com/in/yourprofile)

⭐ If this project demonstrates valuable monitoring and observability skills, please star it!
