
# ELK Set up on Ubunutu

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
    sudo apt install elasticsearch
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

# 3. Installing and Configuring Logstash

Although it’s possible for Beats to send data directly to the Elasticsearch database, it is common to use Logstash to process the data. 
This will allow you more flexibility to collect data from different sources, transform it into a common format, and export it to another database.

    sudo apt install logstash
    sudo nano /etc/logstash/conf.d/02-beats-input.conf

Insert the following input configuration. This specifies a beats input that will listen on TCP port 5044.

    input {
    beats {
        port => 5044
    }
    }

    sudo nano /etc/logstash/conf.d/30-elasticsearch-output.conf

    output {
    if [@metadata][pipeline] {
        elasticsearch {
        hosts => ["localhost:9200"]
        manage_template => false
        index => "%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"
        pipeline => "%{[@metadata][pipeline]}"
        }
    } else {
        elasticsearch {
        hosts => ["localhost:9200"]
        manage_template => false
        index => "%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"
        }
    }
    }

    sudo -u logstash /usr/share/logstash/bin/logstash --path.settings /etc/logstash -t
    sudo systemctl start logstash
    sudo systemctl enable logstash

# 4. Installing and Configuring Filebeat

The Elastic Stack uses several lightweight data shippers called Beats to collect data from various sources and transport them to Logstash or Elasticsearch. Here are the Beats that are currently available from Elastic:

- Filebeat: collects and ships log files.
- Metricbeat: collects metrics from your systems and services.
- Packetbeat: collects and analyzes network data.
- Winlogbeat: collects Windows event logs.
- Auditbeat: collects Linux audit framework data and monitors file integrity.
- Heartbeat: monitors services for their availability with active probing.

In this tutorial we will use Filebeat to forward local logs to our Elastic Stack.

    sudo apt install filebeat
    sudo nano /etc/filebeat/filebeat.yml

In this tutorial, we’ll use Logstash to perform additional processing on the data collected by Filebeat. Filebeat will not need to send any data directly to Elasticsearch, so let’s disable that output. To do so, find the output.elasticsearch section and comment out the following lines by preceding them with a #:

    ...
    #output.elasticsearch:
    # Array of hosts to connect to.
    #hosts: ["localhost:9200"]
    ...

Then, configure the output.logstash section. Uncomment the lines output.logstash: and hosts: ["localhost:5044"] by removing the #. This will configure Filebeat to connect to Logstash on your Elastic Stack server at port 5044, the port for which we specified a Logstash input earlier:

    output.logstash:
    # The Logstash hosts
    hosts: ["localhost:5044"]

    sudo filebeat modules enable system
    sudo filebeat modules list

Next, we need to set up the Filebeat ingest pipelines, which parse the log data before sending it through logstash to Elasticsearch. To load the ingest pipeline for the system module, enter the following command:

    sudo filebeat setup --pipelines --modules system

Next, load the index template into Elasticsearch. An Elasticsearch index is a collection of documents that have similar characteristics. Indexes are identified with a name, which is used to refer to the index when performing various operations within it.
    
    sudo filebeat setup --index-management -E output.logstash.enabled=false -E 'output.elasticsearch.hosts=["localhost:9200"]'

Filebeat comes packaged with sample Kibana dashboards that allow you to visualize Filebeat data in Kibana. Before you can use the dashboards, you need to create the index pattern and load the dashboards into Kibana.
As the dashboards load, Filebeat connects to Elasticsearch to check version information. To load dashboards when Logstash is enabled, you need to disable the Logstash output and enable Elasticsearch output:

    sudo filebeat setup -E output.logstash.enabled=false -E 'output.elasticsearch.hosts=['localhost:9200']' -E setup.kibana.host=localhost:5601
    sudo systemctl start filebeat
    sudo systemctl enable filebeat
    curl -XGET 'http://localhost:9200/filebeat-*/_search?pretty'
