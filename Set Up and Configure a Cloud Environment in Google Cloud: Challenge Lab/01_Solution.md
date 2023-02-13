# Set Up and Configure a Cloud Environment in Google Cloud: Challenge Lab

## GSP321

## Overview
In a challenge lab you’re given a scenario and a set of tasks. Instead of following step-by-step instructions, you will use the skills learned from the labs in the quest to figure out how to complete the tasks on your own! An automated scoring system (shown on this page) will provide feedback on whether you have completed your tasks correctly.

When you take a challenge lab, you will not be taught new Google Cloud concepts. You are expected to extend your learned skills, like changing default values and reading and researching error messages to fix your own mistakes.

To score 100% you must successfully complete all tasks within the time period!

This lab is recommended for students who have completed the labs in the Set up and Configure a Cloud Environment in Google Cloud quest. Are you up for the challenge?

---

## Solution

### Task 1. Create development VPC manually

Requirements:

* Create a VPC called `griffin-dev-vpc` with the following subnets only:
  * `griffin-dev-wp`
    * IP address block: `192.168.16.0/20`
  * `griffin-dev-mgmt`
    * IP address block: `192.168.32.0/20`

#### Steps:

You can use the gcloud command to fast provision this VPC as in:

`gcloud config set compute/region us-east1`

`gcloud config set compute/zone us-east1-b`

`gcloud compute networks create griffin-dev-vpc --subnet-mode=custom`

`gcloud compute networks subnets create griffin-dev-wp --network=griffin-dev-vpc --range=192.168.16.0/20`

`gcloud compute networks subnets create griffin-dev-mgmt --network=griffin-dev-vpc --range=192.168.32.0/20`


---

### Task 2. Create production VPC manually

Requirements:

* Create a VPC called `griffin-prod-vpc` with the following subnets only:
  * `griffin-prod-wp`
    * IP address block: `192.168.48.0/20`
  * `griffin-prod-mgmt`
    * IP address block: `192.168.64.0/20`

#### Steps:

You can use the gcloud command to fast provision this VPC as in:

`gcloud config set compute/region us-east1`

`gcloud config set compute/zone us-east1-b`

`gcloud compute networks create griffin-prod-vpc --subnet-mode=custom`

`gcloud compute networks subnets create griffin-prod-wp --network=griffin-prod-vpc --range=192.168.48.0/20`

`gcloud compute networks subnets create griffin-prod-mgmt --network=griffin-prod-vpc --range=192.168.64.0/20`


---

### Task 3. Create bastion host

Requirements:

* Create a bastion host with two network interfaces, one connected to `griffin-dev-mgmt` and the other connected to `griffin-prod-mgmt`. Make sure you can SSH to the host.

#### Steps:

You can use the gcloud command to fast provision this bastion as in:

`gcloud config set compute/region us-east1`

`gcloud config set compute/zone us-east1-b`

`gcloud compute instances create griffin-bastion1 --machine-type=n1-standard-1 --network-interface=network-tier=PREMIUM,subnet=griffin-dev-mgmt --network-interface=network-tier=PREMIUM,subnet=griffin-prod-mgmt --create-disk=auto-delete=yes,boot=yes,device-name=instance-1,image=projects/debian-cloud/global/images/debian-11-bullseye-v20230206`

`gcloud compute firewall-rules create griffin-dev-vpc-allow-icmp-ssh-rdp --direction=INGRESS --priority=1000 --network=griffin-dev-vpc --action=ALLOW --rules=icmp,tcp:22,tcp:3389 --source-ranges=0.0.0.0/0`

`gcloud compute firewall-rules create griffin-prod-vpc-allow-icmp-ssh-rdp --direction=INGRESS --priority=1000 --network=griffin-prod-vpc --action=ALLOW --rules=icmp,tcp:22,tcp:3389 --source-ranges=0.0.0.0/0`


---

### Task 4. Create and configure Cloud SQL Instance

Requirements:

1. Create a MySQL Cloud SQL Instance called griffin-dev-db in us-east1.

2. Connect to the instance and run the following SQL commands to prepare the WordPress environment:

``` SQL
CREATE DATABASE wordpress;
CREATE USER "wp_user"@"%" IDENTIFIED BY "stormwind_rules";
GRANT ALL PRIVILEGES ON wordpress.* TO "wp_user"@"%";
FLUSH PRIVILEGES;
```

These SQL statements create the worpdress database and create a user with access to the wordpress database.

You will use the username and password in __task 6__.

#### Steps:

You can use the console to provision this SQL instance:

![image](https://user-images.githubusercontent.com/5251806/218290366-c2253a79-5d23-4aae-9302-a9b8ce051068.png)

`gcloud sql connect qwiklabs-demo --user=root --quiet`

Apply SQL configurations.


---

### Task 5. Create Kubernetes cluster

* Create a 2 node cluster (n1-standard-4) called griffin-dev, in the griffin-dev-wp subnet, and in zone us-east1-b.

#### Steps:

You can use the gcloud command to fast provision this cluster as in:

`gcloud config set compute/region us-east1`

`gcloud config set compute/zone us-east1-b`

`gcloud container clusters create --machine-type=n1-standard-1 --num-nodes=2 griffin-dev`


---

### Task 6. Prepare the Kubernetes cluster
1. Use Cloud Shell and copy all files from gs://cloud-training/gsp321/wp-k8s.

The WordPress server needs to access the MySQL database using the username and password you created in task 4.

2. You do this by setting the values as secrets. WordPress also needs to store its working files outside the container, so you need to create a volume.

3. Add the following secrets and volume to the cluster using wp-env.yaml.

4. Make sure you configure the username to wp_user and password to stormwind_rules before creating the configuration.

You also need to provide a key for a service account that was already set up. This service account provides access to the database for a sidecar container.

5. Use the command below to create the key, and then add the key to the Kubernetes environment:

``` bash
gcloud iam service-accounts keys create key.json \
    --iam-account=cloud-sql-proxy@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com
kubectl create secret generic cloudsql-instance-credentials \
    --from-file key.json
```

#### Steps:

`gsutil -m cp -r gs://cloud-training/gsp321/wp-k8s .`

`cd wp-k8s`

`vim wp-env.yaml`

change username and password to stormwind_rules

`gcloud iam service-accounts keys create key.json \
    --iam-account=cloud-sql-proxy@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com`
    
`kubectl create secret generic cloudsql-instance-credentials \
    --from-file key.json`


---

### Task 7. Create a WordPress deployment

Now that you have provisioned the MySQL database, and set up the secrets and volume, you can create the deployment using `wp-deployment.yaml`.

1. Before you create the deployment you need to edit wp-deployment.yaml.

2. Replace YOUR_SQL_INSTANCE with griffin-dev-db's Instance connection name.

3. Get the Instance connection name from your Cloud SQL instance.

4. After you create your WordPress deployment, create the service with wp-service.yaml.

5. Once the Load Balancer is created, you can visit the site and ensure you see the WordPress site installer.
At this point the dev team will take over and complete the install and you move on to the next task.

#### Steps:

`vim wp-deployment.yaml`

Replace `YOUR_SQL_INSTANCE` with __griffin-dev-db's Instance connection name__. (Get the Instance connection name from Cloud SQL instance)

`kubectl create deployment -f wp-deployment.yaml`

`kubectl create deployment -f wp-service.yaml`


---

### Task 8. Enable monitoring

* Create an uptime check for your WordPress development site.

#### Steps:


---

### Task 9. Provide access for an additional engineer

* Create an uptime check for your WordPress development site.

#### Steps:

