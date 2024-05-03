
# Fleet Server and nginx integration
Setup a Fleet Server for the Elasticsearch Stack v8.x.

Pre-requisites: ElasticSearch and Kibana V8.x installation

# 1. Installing and Configuring Fleet  

Before we can install fleet server, we need to instruct Elasticsearch and Kibana of settings the Fleet Sever will use.

Go to Kibana --> Go to Menu --> Go to Fleet.

Go to Settings --> Outputs
Press on Pencil Icon in the Actions column.
Choose Elasticsearch for Type.

Fill out the Hosts with your actual Elasticsearch url, in our case it was https://elastic.bhooopesh-grafana.com:9200.

If you are using publicly signed certificates, you do not need to reference the Certificate Authority. If you are using self-signed certificates, then paste in your Certificate Authority into the Advance YAML configuration field like this:

Pls copy the below certificate to Fleet server as well.

    ssl.certificate_authorities: ["/etc/certs/fleet/ca.crt"]
Press Save and apply settings.

# 2. Configure Fleet Server Host
Add a Fleet Server
Press Advanced.
Press Create policy.
Go to Choose a deployment mode for security.
Choose Production.
Go to Add your Fleet Server host.
Enter fleet (or any other label) for Name and 
https://fleet.bhooopesh-grafana.com:8220 for URL.
Press Add Host.
Press Generate Service Token.
Copy the code to install the Fleet Server:
Add a Fleet Server

You will need to edit the install command, so paste it into a file 

Run the below command in elastic search server for generating certificates:  

    /usr/share/elasticsearch/bin/elasticsearch-certutil cert \
    --out /root/fleet.zip \
    --name fleet \
    --ca-cert /etc/elasticsearch/certs/ca/ca.crt \
    --ca-key /etc/elasticsearch/certs/ca/ca.key  \
    --dns fleet.bhooopesh-grafana.com \
    --pem;
    cd /root
    unzip fleet.zip

Copy the fleet.crt and fleet.key to Fleet server

# 3. Install Fleet Server
Run the below commands on fleet server to install it.

    mkdir /etc/certs/fleet -p
    curl -L -O https://artifacts.elastic.co/downloads/beats/elastic-agent/elastic-agent-8.13.2-linux-x86_64.tar.gz
    tar xzvf elastic-agent-8.13.2-linux-x86_64.tar.gz
    cd elastic-agent-8.13.2-linux-x86_64
    sudo ./elastic-agent install --url=https://fleet.bhooopesh-grafana.com:8220 \
    --fleet-server-es=https://elastic.bhooopesh-grafana.com:9200 \
    --fleet-server-service-token=AAEAAWVsYXN0aWMvZmxlZXQtc2VydmVyL3Rva2VuLTE3MTQyMTY4MjI5NjU6SkJpOHk5UGVSdkNNUTRhbVJNUWhqZw \
    --fleet-server-policy=fleet-server-policy \
    --certificate-authorities=/etc/certs/fleet/ca.crt \
    --fleet-server-es-ca=/etc/certs/fleet/ca.crt \
    --fleet-server-cert=/etc/certs/fleet/fleet.crt \
    --fleet-server-cert-key=/etc/certs/fleet/fleet.key \
    --fleet-server-port=8220

Go back to Kibana and go to Fleet > Agents.
Confirm you see a healthy Fleet Server.

# 4. INSTALL NGINX AND ELASTIC AGENT
Go to the Ubuntu server you want to install Nginx on. Then run this command:

    apt-get install -y nginx
    http://<ip or domain>:80

Go to Kibana, then go to Menu > Integrations and search for Nginx.
Press Add Nginx.
Scroll down to Where to add this integration?.
Choose New hosts.
Press Save and continue.
In the popup, press Save and deploy changes.
Press Add agent.

Copy the linux tar commands

    mkdir /etc/app/certs -p
Copy the certificates in above folder from fleet server

    curl -L -O https://artifacts.elastic.co/downloads/beats/elastic-agent/elastic-agent-8.13.2-linux-x86_64.tar.gz
    tar xzvf elastic-agent-8.13.2-linux-x86_64.tar.gz
    cd elastic-agent-8.13.2-linux-x86_64
    sudo ./elastic-agent install --url=https://fleet.bhooopesh-grafana.com:8220 \
    --enrollment-token=S2hKWUg0OEJTRWQ3UnVTal9BUUQ6dFJVZXlmMmpSc2lDNDFXYXBJeXZOQQ== \
    --fleet-server-es-ca=/etc/app/certs/fleet.crt \
    --certificate-authorities=/etc/app/certs/ca.crt

Confirm success under Fleet > Agents.

# 5. DEBUGGING
If you run into issues, you can find logs for elastic agent in:  

    /opt/Elastic/Agent/data/elastic-agent-<id>/logs








