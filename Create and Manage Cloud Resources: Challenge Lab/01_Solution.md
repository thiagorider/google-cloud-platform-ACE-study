# Create and Manage Cloud Resources: Challenge Lab

## GSP313

## Overview
In a challenge lab you’re given a scenario and a set of tasks. Instead of following step-by-step instructions, you will use the skills learned from the labs in the quest to figure out how to complete the tasks on your own! An automated scoring system (shown on this page) will provide feedback on whether you have completed your tasks correctly.

When you take a challenge lab, you will not be taught new Google Cloud concepts. You are expected to extend your learned skills, like changing default values and reading and researching error messages to fix your own mistakes.

To score 100% you must successfully complete all tasks within the time period!

This lab is recommended for students who have enrolled in the Create and Manage Cloud Resources quest. Are you ready for the challenge?

Topics tested:

* Create an instance

* Create a 3-node Kubernetes cluster and run a simple service

* Create an HTTP(s) load balancer in front of two web servers

---

## Solution

### Task 1. Create a project jumphost instance

Requirements:

* Name the instance Instance name `XYZ` .
* Use an f1-micro machine type.
* Use the default image type (Debian Linux).

#### Steps:

You can use the gcloud command to fast provision this machine as in:
`gcloud config set compute/region us-east1`  (use the same region required by Task 2)

`gcloud config set compute/zone us-east1-d`  (use the same zone required by Task 2)

`gcloud compute instances create XYZ --machine-type f1-micro`

### Task 2. Create a Kubernetes service cluster

The team is building an application that will use a service running on Kubernetes. You need to:

* Create a zonal cluster using us-east1-d .
* Use the Docker container hello-app (gcr.io/google-samples/hello-app:2.0) as a placeholder; the team will replace the container with their own work later.
* Expose the app on port 8081 .

#### Steps:

You can use the gcloud command to fast provision this cluster as in:

`gcloud config set compute/region us-east1`

`gcloud config set compute/zone us-east1-d`

`gcloud container clusters create --machine-type=n1-standard-1 nucleus-cluster`

For deploying the requested app and expose, use the commands:

`gcloud container clusters get-credentials nucleus-cluster`

`kubectl create deployment hello-server --image=gcr.io/google-samples/hello-app:2.0`

`kubectl expose deployment hello-server --type=LoadBalancer --port 8081`

Wait for external ip to be assigned and check your progress.

### Task 3. 


gcloud compute instance-templates create nucleus-template-1 --network=default --subnet=default --tags=allow-health-check --machine-type=n1-standard-1 --image-family=debian-11 --image-project=debian-cloud --metadata=startup-script=\#\!\ /bin/bash$'\n'apt-get\ update$'\n'apt-get\ install\ -y\ nginx$'\n'service\ nginx\ start$'\n'sed\ -i\ --\ \'s/nginx/Google\ Cloud\ Platform\ -\ \'\"\\\$HOSTNAME\"\'/\'\ /var/www/html/index.nginx-debian.html

gcloud compute instance-groups managed create nucleus-lb-backend-group --template=nucleus-template-1 --size=2

gcloud compute firewall-rules create allow-tcp-rule-967 --network=default --action=allow --direction=ingress --target-tags=allow-health-check --rules=tcp:80

gcloud compute addresses create lb-ipv4-1 --ip-version=IPV4 --global

gcloud compute health-checks create http http-basic-check --port 80

gcloud compute backend-services create web-backend-service --protocol=HTTP --port-name=http --health-checks=http-basic-check --global

gcloud compute backend-services add-backend web-backend-service --instance-group=nucleus-lb-backend-group --global

gcloud compute url-maps create web-map-http --default-service web-backend-service

gcloud compute target-http-proxies create http-lb-proxy --url-map web-map-http

gcloud compute forwarding-rules create http-content-rule --address=lb-ipv4-1 --global --target-http-proxy=http-lb-proxy --ports=80

