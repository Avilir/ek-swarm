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

The machines' minimum hardware requirements are : **8 CPU's & 16 GiB of Memory**

You need to have regular user with *sudo* privileges  - I used the default ***centos*** user

### Let's Install Docker on the host(s) ###

**If you use more than one host, this procedure need to be run on all of them**

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

Now, we have docker installed on the system, we need some more things to run the elasticsearch image :

### Let's create a Docker Swarm cluster ###

**On the main hosts :**

Initialize the swarm cluster

    $ docker swarm init --advertise-addr <host ip>

**On the other hosts :**

    $ docker swarm join --token <token> <main host ip>

For redundancy, it is better to have more than one manager, so on one of the other hosts run :

    $ docker node promote

### Final requirements ###

* We need Java, so execute the command

    `$ sudo yum install java`

* We need Git (for cloning this repo.), so execute the command
  
    `$ sudo yum install git`

* At the home directory (of centos user) create the directories for the data

    `$ sudo mkdir -p ~/ES_Kibana/data/elasticsearch`

    `$ sudo mkdir -p ~ES_Kibana/data/kibana/data`
* We need to change permission for this directories
    
    `$ sudo chown -R :root ~ES_Kibana/data`

When we use more than one host in the docker swarm cluster, we need shared storage that all the hosts can use.

on all hosts, make sure that NFS share is mounted on the data directory

    $ sudo vi /etc/fstab

add the next line to the fstab file :

    <nfs server>:<share path>  /home/centos/ES_Kibana/data   nfs defaults 0 0

We need to configure the virtual memory :

    $ sudo sysctl -w vm.max_map_count=262144

For permanent change, update the file `/etc/sysctl.conf` with the value `vm.max_map_count=262144`

Reboot all hosts, and verify that :
* all hosts have the share mounted on `/home/centos/ES_Kibana/data`
* docker services is run on all hosts : `systemctl status docker`
* the virtual memory was set : `sysctl vm.max_map_count`

**Now we are all set**

## Deployment of the Elasticsearch & Kibana ##

Clone this repo into the directory on the main Docker Swarm host

    $ cd ~/ES_Kibana
    $ git clone git@github.com:Avilir/ek-swarm.git .

Deploy the service into the docker swarm

    $ docker stack deploy -c $(pwd)/docker-compose.yml elk

Verify that the cluster is running :

    $ docker service ls

Sample output

    ID             NAME                MODE         REPLICAS   IMAGE                                                  PORTS
    aaaaaaaaaaaa   elk_elasticsearch   replicated   1/1        docker.elastic.co/elasticsearch/elasticsearch:7.16.3   *:9200->9200/tcp, *:9300->9300/tcp
    bbbbbbbbbbbb   elk_kibana          replicated   1/1        docker.elastic.co/kibana/kibana:7.16.3                 *:5601->5601/tcp

Now when the elasticsearch is up and running, it can be scaled for HA by updating the replication count

    $ docker service update --replicas=2 <elasticsearch service id>

Note: **elasticsearch replica count can be changed only after the one replica is run**

