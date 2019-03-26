# 366-elk
First get a domain from byu domains
In the dns zone editor create an "A record" for your public ip address.

## Installation:
`
sudo apt-get update && sudo apt-get upgrade
`

### JDK
If you don't have JDK you will need to install it.

`
sudo apt-get install default-jdk
`

### Nginx:
`
sudo apt-get install nginx
`

### Elasticsearch:
`
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
echo "deb https://artifacts.elastic.co/packages/6.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-6.x.list
sudo apt-get update
sudo apt-get install elasticsearch
`

### Install kibana:
`
sudo apt-get install kibana
`

## Settin up Nginx:
Run the following command replacing bracketed [] text with your own values.

``
echo "[insert username]:`openssl passwd -apr1`" | sudo tee -a /etc/nginx/htpasswd.users
sudo vim /etc/nginx/sites-available/[your domain]
``

You will need to add the following code block into "/etc/nginx/sites-available/[your domain]",
dont forget to replace [] text with your own values:

`
    server {
    listen 80;

    server_name [example.com];

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
`

Run the following command to create a link from sites enable to sites available and restart Nginx:

`
sudo ln -s /etc/nginx/sites-available/[your domain] /etc/nginx/sites-enabled/[your domain]
sudo systemctl restart nginx
`

## Install the remaining services:
`
    sudo apt-get install logstash
    sudo apt-get install filebeat
    sudo apt-get install auditbeat
    sudo apt-get install metricbeat
    sudo filebeat modules enable elasticsearch logstash kibana system
    sudo metricbeat modules enable elasticsearch logstash kibana
    cd /usr/share/elasticsearch
    sudo bin/elasticsearch-plugin install ingest-geoip
    sudo bin/elasticsearch-plugin install ingest-user-agent
`

## Config files
All of the config files need to be copied from this repository into their respectful places

For conf files not in conf.d folder place them according to the guidelines below*:
/etc/[servicename]/[YAML file]
e.g. /etc/kibana/kibana.yml

Place files in the conf.d folder into /etc/logstash/conf.d/

* You can also pull them all as a repository down into a single folder and link them using
`
ln -s [path of file] [path where it belongs]
`
This would allow you to use a repo to manage the code easily as all you would have to do is pull any updates
and not worry about moving the files to where they belong.

## Install Templates and Dashboards

### Start Elastic Stack:
`
sudo systemctl start elasticsearch logstash kibana
`

### Install templates and dashboards:
The first threee steps will install templates to logstash.

`
sudo filebeat setup --template -E output.logstash.enabled=false -E output.elasticsearch.enabled=true -E 'output.elasticsearch.hosts=["localhost:9200"]'
sudo metricbeat setup --template -E output.logstash.enabled=false -E output.elasticsearch.enabled=true -E 'output.elasticsearch.hosts=["localhost:9200"]'
sudo auditbeat setup --template -E output.logstash.enabled=false -E output.elasticsearch.enabled=true -E 'output.elasticsearch.hosts=["localhost:9200"]'
`

The next three commands will set the dashboards up on Kibana

`
sudo filebeat setup --dashboards
sudo metricbeat setup --dashboards
sudo auditbeat setup --dashboards
`

## Start Beats
To start your beats run the following command.
`
sudo systemctl start filebeat metricbeat auditbeat
`

To make them all persistent run:
`
sudo systemctl enable elasticsearch logstash kibana filebeat metricbeat logstash
`

## Troubleshooting
### General
If you run into any prolems while setting this up you can use the journalctl command as follows:
`
journalctl -u [name of service]

e.g.
journalctl -u nginx.service
`

### Beats
If you run into prolems with any of the beats you can also use the "-e" flag
`
e.g. filebeat -e
`
Which will output the logs to the console.

### Common problems
Almost all the problems I had were with conf files, if you don't clone this repo but create configs
of your own make sure everything is formatted correctly, most improtantly make sure [] and {} match.
These are fatal errors if they do not match.


## My URL
timothymoffett.com
