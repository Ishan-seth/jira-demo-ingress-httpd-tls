# jira-demo-ingress-httpd-tls

A demo JIRA deployment to Azure AKS including nginx ingress controller (for https/tls) and apache httpd web server (which improves the performance of the application).

- NGINX Ingress 
- Apache httpd web server
- Single-node Jira server 
- Postgres DB server

Created by: Neil Brown - Email:  neil.n.brown@gmail.com

JIRA & Postgres DB Images provided by: Edison Wong, PantaRei Design: https://github.com/alvistack

## Overview

- This setup includes ingress & httpd which makes it more suitable as a starting point to create production-ready JIRA instance.
- For a more simplfied demo (without ingress and httpd) please see: https://github.com/neil-n-brown/jira-demo.


## What Do We Need?

- Nginx Ingress: https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-0.32.0/deploy/static/provider/cloud/deploy.yaml
- 1 namespace:  00-devops-namespace.yml
- Default Azure Storage Classes + 1 custom Storage Class for `var-lib-postgresql-data`. Custom storage class is required for Postgres as the default mount options for AzureDisk do not allow the Postgres user to write to the location.
- Persistent Volume Claims for `var-lib-postgresql-data` (pvc-postgres-data.yaml) `var-atlassian-application-data-jira` (pvc-jira-data.yaml) and `var-atlassian-application-data-jira-shared` (pvc-jira-shared.yaml) directories.
- 1 Secret for Postgres DB: secret-postgres.yml
- 1 StatefulSet for the Jira server: sts-jira.yml
- 1 StatefulSet for the PostgresSQL DB server: sts-postgres.yml
- 1 Headless Service for JIRA: svc-jira.yaml
- 1 Headless Service for Postgres: svc-postgres.yml
- 1 Load Balancer for JIRA Cluster:  svc-jira-cluster.yml
- 1 Load Balancer for Postgres Cluser svc-postgres-cluster.yaml


## Usage


### Deploy JIRA to AKS Cluster

0) Deploy Nginx Ingress: 

    kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-0.32.0/deploy/static/provider/cloud/deploy.yaml
    
    If you wish to use Internal IP for Ingress refer to:  https://docs.microsoft.com/en-us/azure/aks/ingress-internal-ip

1) Ensure you are logged on to cluster:  az aks get-credentials --resource-group YourResourceGroup --name YourAKSCluster

2) git clone https://github.com/neil-n-brown/jira-demo-ingress-httpd-tls.git >>> repo to your local machine

3) cd jira-demo-ingress-httpd-tls  >>> Switch to the repo folder on your local machine

4) Edit the following files:

    00-devops-namespace.yml >>> If you wish to edit all your files to use a different namespace (default is "devops") then please note that Namespace must be created first to deploy the other configurations. Ensure namespace yaml filename is the first file that will be read when you run the kubectl apply -f command. e.g. add "00-" to beginning of yaml filename

    svc-jira-cluster.yml >>> Decide if you wish to you Public IP or Internal Loadbalancer (see comments in the file for detais)

    ingress-nginx.yml >>> Edit with your own cert details. To get the nginx controller up and running without https then comment out the highlighted lines and change host to host: *

5) kubectl apply -Rf . >>> This command will apply all of your YAML files in the current directory

6) kubectl -n devops get all  >>> This command will display all statefulsets, pods and services. You want to see all pods in a "running state 1/1" this may take several minutes while the images are downloaded etc.

7) Create DNS name and link it to the ip address of the NGINX Ingress Controller

8) Create tls crt and key then convert them to a kubernetes secret (example on Windows 10 Git bash below - please remember to use your own details for /CN = yourdomainname and /O = your crt/key name)

    openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
        -out jira-demo-ingress-tls.crt \
        -keyout jira-demo-ingress-tls.key \
        -subj "//skip=yes/CN=jira-demo.japaneast.cloudapp.azure.com/O=jira-demo-ingress-tls"

    kubectl create secret tls jira-demo-ingress-tls \
        --namespace ingress-nginx \
        --key jira-demo-ingress-tls.key \
        --cert jira-demo-ingress-tls.crt    


### Configure Jira

Visit the DNS name you configured earlier https://yourdomainname.com 

At this point you follow the setup wizard to connect the application to the Postgres DB and a Trial License from Atlassian.
