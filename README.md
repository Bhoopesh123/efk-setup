
# EFK Set up on Minikube Cluster  

Elasticsearch will be installed first as a Statefulset which will store all the data as indexed
Fluend will be installed as daemonset so that logs can be captured from all Nodes
Kibana will be installed as deployment so that it can invokes the Elasticsearch Server and run all the dashboards.

# 1. Install Elasticsearch on Minikube Cluster:  

Follow the below steps to install it on Cluster:

    cd elasticsearch
    kubectl create -f statefulset.yaml
    kubectl create -f service.yaml

# 2. Install Kibana on Minikube Cluster:  

Follow the below steps to install it on Cluster:

    cd kibana
    kubectl create -f deployment.yaml
    kubectl create -f service.yaml

# 3. Install Fluentd on Minikube Cluster:  

Follow the below steps to install it on Cluster:

    cd fluentd
    kubectl create -f clusterrole.yaml
    kubectl create -f serviceaccount.yaml
    kubectl create -f clusterrolebinding.yaml
    kubectl create -f daemonset.yaml

# 3. Validate the EFK Cluster  
Run one pod in the same namespace to capture the logs on cluster  

    kubectl run nginx --image=nginx --restart=Never
    kubectl run mycurlpod --image=curlimages/curl -i --tty -- sh

