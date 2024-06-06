
# 1. ELK Setup in Ubuntu Machine
Reference Documenation:  

https://github.com/Bhoopesh123/efk-setup/blob/main/README_ELK_Ubuntu.md


# 2. Using Logstash Email Plugin

Reference Documentation: 

https://www.elastic.co/guide/en/logstash/current/plugins-outputs-email.htmlion.html

    cd /etc/logstash/conf.d
    vim 30-elasticsearch-output.conf

Add the below content in the configuration file if not present. 

    filter {
    if "filebeat" in [message] {
            mutate { add_tag => "bhoopeshsharma" }
    }
    }

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

    if "bhoopeshsharma" in [tags] {
        stdout { }
        email {
        to => "bhoopeshsharmal306@gmail.com"
        from => "logstash@gmail.com"
        subject => "Test Alert at %{@timestamp}"
        body => "Tags: %{tags}"
        via=> "smtp"
        address => "smtp.gmail.com"
        port => 587
        authentication => "plain"
        use_tls => true
        username => "bhoopeshsharmal306@gmail.com"
        password => "***********"
        }
    }
    }

# 2. Verify the Email box with Alerts notifications 

Run the logstash with below commands.  

    cd /etc/logstash/conf.d
    sudo cp -p 30-elasticsearch-output.conf /usr/share/logstash 
    sudo systemctl status logstash
    sudo systemctl stop logstash
    sudo systemctl start logstash
    sudo systemctl status logstash

Verify the logstash logs from below location for any errors. 

    cd /var/log/logstash
    tail -f logstash-plain.log

Check the tags on Kibana and Alerts in Gmail. 