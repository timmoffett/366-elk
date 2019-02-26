# 366-elk
First get a domain from byu domains
In the dns zone editor create an "A record" for your public ip address.

Installation:
sudo apt-get update && sudo apt-get upgrade

JDK
sudo apt-get install default-jdk

nginx:
sudo apt-get install nginx

elasticsearch:
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
echo "deb https://artifacts.elastic.co/packages/6.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-6.x.list
sudo apt-get update
sudo apt-get install elasticsearch

kibana:
sudo apt-get install kibana

setup nginx:
echo "[insert username]:`openssl passwd -apr1`" | sudo tee -a /etc/nginx/htpasswd.users
sudo vim /etc/nginx/sites-available/[your domain]
add the following code block:
"""
server {
    listen 80;

    server_name example.com;

    auth_basic "Restricted Access";
    auth_basic_user_file /etc/nginx/htpasswd.users;

    location / {
        proxy_pass http://localhost:5601;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
"""
sudo ln -s /etc/nginx/sites-available/[your domain] /etc/nginx/sites-enabled/[your domain]
sudo systemctl restart nginx

install remaining services:
sudo apt-get install logstash
sudo apt-get install filebeat
sudo apt-get install auditbeat
sudo apt-get install metricbeat
sudo filebeat modules enable elasticsearch logstash kibana system
sudo metricbeat modules enable elasticsearch logstash kibana
cd /usr/share/elasticsearch
sudo bin/elasticsearch-plugin install ingest-geoip
sudo bin/elasticsearch-plugin install ingest-user-agent

place config files in their rightful places
/etc/[servicename/[YAML file]
e.g. /etc/kibana/kibana.yml

place files in the conf.d folder into /etc/logstash/conf.d/

start elk stack:
sudo systemctl start elasticsearch logstash kibana

install templates and dashboards:
sudo filebeat setup --template -E output.logstash.enabled=false -E output.elasticsearch.enabled=true -E 'output.elasticsearch.hosts=["localhost:9200"]'
sudo metricbeat setup --template -E output.logstash.enabled=false -E output.elasticsearch.enabled=true -E 'output.elasticsearch.hosts=["localhost:9200"]'
sudo auditbeat setup --template -E output.logstash.enabled=false -E output.elasticsearch.enabled=true -E 'output.elasticsearch.hosts=["localhost:9200"]'
sudo filebeat setup --dashboards
sudo metricbeat setup --dashboards
sudo auditbeat setup --dashboards

start beats:
sudo systemctl start filebeat metricbeat logstash

make them all persistent:
sudo systemctl enable elasticsearch logstash kibana filebeat metricbeat logstash

