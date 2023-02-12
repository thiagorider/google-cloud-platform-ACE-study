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


### Task 3. Create bastion host

Requirements:

* Create a bastion host with two network interfaces, one connected to `griffin-dev-mgmt` and the other connected to `griffin-prod-mgmt`. Make sure you can SSH to the host.

#### Steps:

You can use the gcloud command to fast provision this VPC as in:
`gcloud config set compute/region us-east1`

`gcloud config set compute/zone us-east1-b`

`gcloud compute instances create griffin-bastion1 --machine-type=n1-standard-1 --network-interface=network-tier=PREMIUM,subnet=griffin-dev-mgmt --network-interface=network-tier=PREMIUM,subnet=griffin-prod-mgmt --create-disk=auto-delete=yes,boot=yes,device-name=instance-1,image=projects/debian-cloud/global/images/debian-11-bullseye-v20230206`

`gcloud compute firewall-rules create griffin-dev-mgmt-vpc-allow-icmp-ssh-rdp --direction=INGRESS --priority=1000 --network=griffin-dev-mgmt --action=ALLOW --rules=icmp,tcp:22,tcp:3389 --source-ranges=0.0.0.0/0`

`gcloud compute firewall-rules create griffin-prod-mgmt-vpc-allow-icmp-ssh-rdp --direction=INGRESS --priority=1000 --network=griffin-prod-mgmt --action=ALLOW --rules=icmp,tcp:22,tcp:3389 --source-ranges=0.0.0.0/0`

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
