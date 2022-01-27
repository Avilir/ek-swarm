# ek-swarm
Deployment of Elastic-Search and Kibana using docker swarm

## Introduction
### What are Elasticsearch and Kibana
#### Elasticsearch

*from https://www.elastic.co/what-is/elasticsearch :*

    "Elasticsearch is a distributed, free and open search and analytics engine for all types of data,
    including textual, numerical, geospatial, structured, and unstructured. Elasticsearch is built
    on Apache Lucene and was first released in 2010 by Elasticsearch N.V. (now known as Elastic)."

#### Kibana

*from : https://www.elastic.co/what-is/kibana :*

    "Kibana is an free and open frontend application that sits on top of the Elastic Stack, 
    providing search and data visualization capabilities for data indexed in Elasticsearch."

### Deployment method

One method to deploy the EK (***elasticsearch & kibana***) is as Docker containers.

For more robust (scale up) and reliability I chose to deploy it not as a single container on single
host, rather as containers on a ***Docker Swarm*** cluster, and in this manuel I will explain how to
do it.


## Prerequisite

For use a ***Docker Swarm*** cluster I used 2 virtual machines (can be done with bare-metal as well),
with **CentOS-7** Linux installed on them.

You need to have regular user with *sudo* privileges  - I used the default ***centos*** user

First, let’s update the package database:

    $ sudo yum check-update
 
Now run this command. It will add the official Docker repository, download the latest version of Docker, and install it:

    $ curl -fsSL https://get.docker.com/ | sh
 
After installation has completed, start the Docker daemon:

    $ sudo systemctl start docker
 
Verify that it’s running:

    $ sudo systemctl status docker
 
The output should be similar to the following, showing that the service is active and running:

    Output
    ● docker.service - Docker Application Container Engine
       Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
       Active: active (running) since Sun 2016-05-01 06:53:52 CDT; 1 weeks 3 days ago
         Docs: https://docs.docker.com
     Main PID: 749 (docker)
Lastly, make sure it starts at every server reboot:

    $ sudo systemctl enable docker
 
By default, running the docker command requires root privileges.
If you want to avoid typing sudo whenever you run the docker command, add your username to the
docker group:

    $ sudo usermod -aG docker <username>

