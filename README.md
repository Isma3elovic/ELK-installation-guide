# ELK-installation-guide
This guide explains step-by-step how to install **Elasticsearch**, **Logstash**, and **Kibana** (ELK Stack) on an Ubuntu server.
# ELK Stack Installation on Ubuntu Server

This guide explains step-by-step how to install and configure the **ELK Stack** — Elasticsearch, Logstash, and Kibana — on an Ubuntu server.

---

## Prerequisites
- Ubuntu Server (20.04/22.04 recommended)
- Sudo privileges
- Basic knowledge of Linux terminal commands

---

## Installation Steps

```bash
# 1. Update System Packages
sudo apt update
sudo apt upgrade -y
```
# 2. Install Java 17
sudo apt install openjdk-17-jre-headless -y
java -version

# 3. Install Elasticsearch
curl -fsSL https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elastic.gpg
echo "deb [signed-by=/usr/share/keyrings/elastic.gpg] https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-7.x.list
sudo apt update
sudo apt install elasticsearch -y

# Configure Elasticsearch
sudo bash -c 'cat > /etc/elasticsearch/elasticsearch.yml <<EOF
network.host: 0.0.0.0
http.port: 9200
node.name: node-1
discovery.type: single-node
EOF'

# Start and enable Elasticsearch
sudo systemctl start elasticsearch
sudo systemctl enable elasticsearch
sudo systemctl status elasticsearch

# Verify Elasticsearch
curl -X GET "http://localhost:9200"

# 4. Install Logstash
sudo apt install logstash -y

# Configure Logstash
sudo bash -c 'cat > /etc/logstash/conf.d/logstash.conf <<EOF
input {
  beats {
    port => 5044
    host => "0.0.0.0"
  }
}
filter {
  grok {
    match => { "message" => "%{TIMESTAMP_ISO8601:log_timestamp} %{LOGLEVEL:log_level} %{GREEDYDATA:log_message}" }
  }
}
output {
  elasticsearch {
    hosts => ["http://localhost:9200"]
    index => "logs-%{+YYYY.MM.dd}"
  }
  stdout { codec => rubydebug }
}
EOF'

# Start and enable Logstash
sudo systemctl start logstash
sudo systemctl enable logstash
sudo systemctl status logstash

# 5. Install Kibana
sudo apt install kibana -y

# Configure Kibana
sudo bash -c 'cat > /etc/kibana/kibana.yml <<EOF
server.port: 5601
server.host: "0.0.0.0"
EOF'

# Start and enable Kibana
sudo systemctl restart kibana
sudo systemctl enable kibana
sudo systemctl status kibana

# 6. Access the ELK Stack
# Elasticsearch: http://<server-ip>:9200
# Kibana: http://<server-ip>:5601
