
# 1. MSSQL Installation and Configuration
This guide describes how to get started quickly installing MSSQL on Linux RHEL9 machine.  
Reference Documenation: 

https://learn.microsoft.com/en-us/sql/linux/quickstart-install-connect-red-hat?view=sql-server-ver16&tabs=rhel9


    sudo yum install -y mssql-server-selinux
    sudo curl -o /etc/yum.repos.d/mssql-server.repo https://packages.microsoft.com/config/rhel/9/mssql-server-2022.repo
    sudo yum install -y mssql-server
    sudo /opt/mssql/bin/mssql-conf setup
    systemctl status mssql-server
    sudo firewall-cmd --zone=public --add-port=1433/tcp --permanent
    sudo firewall-cmd --reload
    curl https://packages.microsoft.com/config/rhel/9/prod.repo | sudo tee /etc/yum.repos.d/mssql-release.repo
    sudo yum remove mssql-tools unixODBC-utf16 unixODBC-utf16-devel
    sudo yum install -y mssql-tools18 unixODBC-devel
    echo 'export PATH="$PATH:/opt/mssql-tools18/bin"' >> ~/.bash_profile
    echo 'export PATH="$PATH:/opt/mssql-tools18/bin"' >> ~/.bashrc
    source ~/.bashrc

Connect to SQL DB  

    sqlcmd -S localhost -U sa -P 'Jan@2020' -No

    CREATE DATABASE TestDB;
    SELECT Name from sys.databases;
    GO
    USE TestDB;
    CREATE TABLE dbo.Inventory (
        id INT,
        name NVARCHAR(50),
        quantity INT,
        PRIMARY KEY (id)
    );
    INSERT INTO dbo.Inventory VALUES (1, 'banana', 150);
    INSERT INTO dbo.Inventory VALUES (2, 'orange', 154);
    GO
    SELECT * FROM dbo.Inventory
    WHERE quantity > 152;
    GO
    QUIT


# 2. Installing and Configuring MetricBeat

Reference Documentation: 
https://www.elastic.co/guide/en/beats/metricbeat/current/metricbeat-installation-configuration.html

    curl -L -O https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-8.13.1-linux-x86_64.tar.gz
    tar xzvf metricbeat-8.13.1-linux-x86_64.tar.gz
    cd metricbeat-8.13.1-linux-x86_64

    vi metricbeat.yml

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

    cd /home/ec2-user/metricbeat-8.13.1-linux-x86_64/modules.d 
    vi mssql.yml 

    - module: mssql
      metricsets:
        - "transaction_log"
        - "performance"
      hosts: ["sqlserver://localhost"]
      username: sa
      password: Jan@2020
      period: 10s

Enable Metricbeat

    sudo chown root metricbeat.yml 
    sudo chown root modules.d/mssql.yml 
    sudo ./metricbeat -e &

# 3. Installing and Configuring FileBeat

Reference Documentation:  
https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-installation-configuration.html

    curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-8.13.4-linux-x86_64.tar.gz
    tar xzvf filebeat-8.13.4-linux-x86_64.tar.gz

    vi filebeat.yml

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

    ./filebeat modules list
    ./filebeat modules enable mssql
    cd /home/ec2-user/filebeat-8.13.4-linux-x86_64/modules.d 
    vi mssql.yml 

    - module: mssql
      # Fileset for native deployment
      log:
        enabled: true
        # Set custom paths for the log files. If left empty,
        # Filebeat will choose the paths depending on your OS.
        var.paths: ["/var/opt/mssql/log/*.log*"]
    
    ./filebeat setup -e
    sudo chown root filebeat.yml 
    sudo chown root modules.d/mssql.yml 
    sudo ./filebeat -e

# 4. Visualize the Dashboards:
Go to dashboards section and see below dashboards: 

1. [Metricbeat MSSQL] Transaction log ECS
2. [Metricbeat MSSQL] Performance ECS
