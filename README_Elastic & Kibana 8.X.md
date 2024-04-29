
# Elastic and Kibana 8.X Set up on Ubunutu

The Elastic Stack — formerly known as the ELK Stack — is a collection of open-source software produced by Elastic which allows you to search, analyze, and visualize logs generated from any source in any format, a practice known as centralized logging. 

- Elasticsearch: A distributed RESTful search engine which stores all of the collected data.
- Kibana: A web interface for searching and visualizing logs.


# 1. Installing and Configuring Elasticsearch  

Please run the below commands with sudo permissions:  

    apt-get update && apt dist-upgrade -y && apt-get install -y vim curl gnupg gpg
    wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg;
    echo 'deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main' | sudo tee /etc/apt/sources.list.d/elastic-8.x.list;
    apt-get install -y apt-transport-https;
    apt-get install -y elasticsearch;

    The generated password for the elastic built-in superuser is : cnOi+uBX678UvG=5vEZU

    Reset the password of the elastic built-in superuser with
    /usr/share/elasticsearch/bin/elasticsearch-reset-password -u elastic

    Generate an enrollment token for Kibana instances with
    /usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s kibana


# 2. Create Self Signed Certificate for ElasticSearch  

    /usr/share/elasticsearch/bin/elasticsearch-certutil ca --pem --out /etc/elasticsearch/certs/ca.zip  
    cd /etc/elasticsearch/certs/
    unzip ca.zip

Now let's make a self-signed certificate for elastic.bhooopesh-grafana.com and sign it with our ca.crt:  

    /usr/share/elasticsearch/bin/elasticsearch-certutil cert \
    --out /etc/elasticsearch/certs/elastic.zip \
    --name elastic \
    --ca-cert /etc/elasticsearch/certs/ca/ca.crt \
    --ca-key /etc/elasticsearch/certs/ca/ca.key \
    --dns elastic.bhooopesh-grafana.com \
    --pem;

    cd /etc/elasticsearch/certs/;
    unzip elastic.zip

# 3. Configure Elasticsearch  

Go to the /etc/elasticsearch/elasticsearch.yml file. Edit the following fields:

    ...etc...
    cluster.name: es-demo
    http.port: 9200
    network.host: 0.0.0.0
    ...etc...
    xpack.security.http.ssl:
        enabled: true
        key: certs/elastic/elastic.key
        certificate: certs/elastic/elastic.crt
        certificate_authorities: certs/ca/ca.crt
    ...etc...

    chown -R elasticsearch:elasticsearch /etc/elasticsearch

    systemctl enable elasticsearch;
    systemctl start elasticsearch;
    systemctl status elasticsearch;


    curl -X GET -u elastic:cnOi+uBX678UvG=5vEZU https://elastic.bhooopesh-grafana.com:9200 --cacert /etc/elasticsearch/certs/ca/ca.crt

# 4. Installing and Configuring the Kibana   

    wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg;
    echo 'deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main' | sudo tee /etc/apt/sources.list.d/elastic-8.x.list;
    apt-get install -y apt-transport-https;
    apt-get install -y kibana;

# 5. Create Self Signed Certificate for Kibana 

Run the following commands on elastic server

    /usr/share/elasticsearch/bin/elasticsearch-certutil cert \
    --out /etc/elasticsearch/certs/kibana.zip \
    --name kibana \
    --ca-cert /etc/elasticsearch/certs/ca/ca.crt \
    --ca-key /etc/elasticsearch/certs/ca/ca.key \
    --dns kibana.bhooopesh-grafana.com \
    --pem;

    cd /etc/elasticsearch/certs/;
    unzip kibana.zip

We will copy the /etc/elasticsearch/certs/kibana/* over to the kibana server kibana.bhooopesh-grafana.com.

# 6. Configure Kibana

    mkdir /etc/kibana/certs/kibana/ca -p

Copy the certs from elastic server /etc/elasticsearch/certs/kibana/* to kibana server /etc/kibana/certs/kibana
Go to the /etc/kibana/kibana.yml file. Edit the following fields:

    server.port: 5601
    server.host: 0.0.0.0
    server.publicBaseUrl: "https://kibana.bhooopesh-grafana.com:5601"
    server.ssl.enabled: true
    server.ssl.key: /etc/kibana/certs/kibana/kibana.key
    server.ssl.certificate: /etc/kibana/certs/kibana/kibana.crt
    server.ssl.certificateAuthorities: /etc/kibana/certs/kibana/ca/ca.crt
    elasticsearch.hosts: ["https://elastic.bhooopesh-grafana.com:9200"]
    elasticsearch.ssl.verificationMode: full
    elasticsearch.ssl.certificateAuthorities: ["/etc/kibana/certs/kibana/ca/ca.crt"]

CREATE SERVICE TOKEN
Run this command on the Elasticsearch server:  

    /usr/share/elasticsearch/bin/elasticsearch-service-tokens create elastic/kibana kibana-token 

SERVICE_TOKEN elastic/kibana/kibana-token = AAEAAWVsYXN0aWMva2liYW5hL2tpYmFuYS10b2tlbjpkd2lnUVJQdlRveS11YTh5TmNBWjN3

    chown -R elasticsearch:elasticsearch /etc/elasticsearch
    chown -R elasticsearch:elasticsearch service_tokens
    systemctl restart elasticsearch

Copy the token that you see.

Run this command on the Kibana server:  

    /usr/share/kibana/bin/kibana-keystore add elasticsearch.serviceAccountToken

Paste in the token after the prompt.
    chown -R kibana:kibana ./

# 7. Start Kibana

    systemctl enable kibana;
    systemctl start kibana;
    loginid: elastic:
    password: cnOi+uBX678UvG=5vEZU

# 8. Install MetricBeat on Application Server

    curl -L -O https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-8.13.1-linux-x86_64.tar.gz
    tar xzvf metricbeat-8.13.1-linux-x86_64.tar.gz
    cd metricbeat-8.13.1-linux-x86_64

# 9. Configure MetricBeat on Application Server

Go to metricbeat.yml and edit it.  

    setup.kibana:
        host: "https://kibana.bhooopesh-grafana.com:5601"
        protocol: "https"
        ssl:
            enabled: true
            key: /etc/elasticsearch/certs/elastic/kibana.key
            certificate: /etc/elasticsearch/certs/elastic/kibana.crt
            certificate_authorities: /etc/elasticsearch/certs/ca/ca.crt

    output.elasticsearch:
        hosts: ["https://elastic.bhooopesh-grafana.com:9200"]
        preset: balanced
        protocol: "https"
        username: "elastic"
        password: "cnOi+uBX678UvG=5vEZU"
        ssl:
            enabled: true
            key: /etc/elasticsearch/certs/elastic/elastic.key
            certificate: /etc/elasticsearch/certs/elastic/elastic.crt
            certificate_authorities: /etc/elasticsearch/certs/ca/ca.crt

Install nginx

    apt-get install -y nginx

Enable Modules  
    ./metricbeat modules enable nginx
    sudo chown root metricbeat.yml 
    sudo chown root modules.d/nginx.yml 
    sudo ./metricbeat -e