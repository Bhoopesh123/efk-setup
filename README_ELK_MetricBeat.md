
# ELK Set up with MetricBeat on Ubunutu

The Elastic Stack — formerly known as the ELK Stack — is a collection of open-source software produced by Elastic which allows you to search, analyze, and visualize logs generated from any source in any format, a practice known as centralized logging. 

The Elastic Stack has four main components:

- Elasticsearch: A distributed RESTful search engine which stores all of the collected data.
- Logstash: The data processing component of the Elastic Stack which sends incoming data to Elasticsearch.
- Kibana: A web interface for searching and visualizing logs.
- Beats: Lightweight, single-purpose data shippers that can send data from hundreds or thousands of machines to either Logstash or Elasticsearch.

# 1. Installing and Configuring Elasticsearch  

Please run the below commands with sudo permissions:

    curl -fsSL https://artifacts.elastic.co/GPG-KEY-elasticsearch |sudo gpg --dearmor -o /usr/share/keyrings/elastic.gpg
    echo "deb [signed-by=/usr/share/keyrings/elastic.gpg] https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list
    sudo apt update
    sudo apt-get install elasticsearch
    sudo nano /etc/elasticsearch/elasticsearch.yml
    Change the network.host to "localhost"
    sudo systemctl start elasticsearch
    sudo systemctl enable elasticsearch
    curl -X GET "localhost:9200"

# 2. Installing and Configuring the Kibana Dashboard  

According to the official documentation, you should install Kibana only after installing Elasticsearch. Installing in this order ensures that the components each product depends on are correctly in place.

    sudo apt install kibana
    sudo systemctl enable kibana
    sudo systemctl start kibana

The following command will create the administrative Kibana user and password, and store them in the htpasswd.users file. You will configure Nginx to require this username and password and read this file momentarily:

    echo "kibanaadmin:`openssl passwd -apr1`" | sudo tee -a /etc/nginx/htpasswd.users

# 3. Installing and Configuring MetricBeat  
To Install Metricbeat, please run the following command.  

    sudo apt install metricbeat

Start and enable Metricbeat auto start after a server reboot, using the following commands.  

    sudo systemctl enable metricbeat.service
    sudo systemctl start metricbeat.service
    sudo systemctl status metricbeat.service

Navigate to the Metricbeat configuration directory.  

    cd /etc/metricbeat

Update the metricbeat.yml with the following configurations in output. elasticsearch section.

    hosts : [“<your_server_ip>/localhost:9200”]
    protocol: “https”
    username: "elastic" 
    password : <superuser_password>
    ssl.certificate_authority : [“ your_certificate_path” ]

To Setup the default metricbeat dashboard on Kibana, please execute the following command. 

    sudo metricbeat setup --dashboards  

Loading dashboards (Kibana must be running and reachable)
Loaded dashboards

To verify the dashboard created from the above step, Please login into Kibana.
Navigate to the dashboard section  

    Kibana > Stack Management > Index Management > Data Streams

Search for Metricbeat Dashboards, if the Metricbeat dashboard is loaded successfully, the following dashboards will appear on the screen.  

    Kibana > Dashboards

# 4. Configuring System Metrics
Go to Modules Folder

    cd /etc/metricbeat/modules.d
    mv system.yml.disabled system.yml
    sudo vim system.yml

Paste the contents of system.yml contents from /efk-setup/metricbeat/system.yml to /etc/metricbeat/modules.d/system.yml 

Restart the metric beat service   

    sudo systemctl stop metricbeat.service
    sudo systemctl start metricbeat.service
    sudo systemctl status metricbeat.service

# 5. Installing and Configuring Node Exporter Metrics

Before you process this make sure node exporter is running on port 9100. 
Post that go to modules folder   

    cd /etc/metricbeat/modules.d
    mv prometheus.yml.disabled prometheus.yml
    sudo vim prometheus.yml

Paste the contents of prometheus.yml contents from /efk-setup/metricbeat/prometheus.yml to /etc/metricbeat/modules.d/prometheus.yml

Restart the metric beat service  

    sudo systemctl stop metricbeat.service
    sudo systemctl start metricbeat.service
    sudo systemctl status metricbeat.service

# 6. Add ElasticSearch Datasource in Grafana

Login to Grafana 

Go to Home --> Connections --> Data sources --> Elasticsearch 

Add this datasource 

Give a name "Elasticsearch-Dev" 

URL: http://localhost:9200 

Click Save and test if no authentication 

