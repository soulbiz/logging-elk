# Installation

We'll be covering the installation process for a Fedora 23 server.
Anyway, it should provide enough guidance to apply it in any other suitable distribution.

## Prerequisites

For our *ELK Server* (from now on, **ELKS**) we'll also need to install:

* Java 8 (As recommended by Elasticsearch ATM, or the latest version)
* Nginx (To setup a reverse proxy for external connections)

We will not cover that here, since we assume it should be easily done!

## Install Elasticsearch

Elasticsearch can be installed with a package manager by adding Elastic's package repository.

Run the following command to import the Elasticsearch public GPG key into rpm:

    sudo rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch

Create a new yum repository file for Elasticsearch.

    sudo vim /etc/yum.repos.d/elasticsearch.repo

Add the following repository configuration:
/etc/yum.repos.d/elasticsearch.repo

	[elasticsearch-5.x]
	name=Elasticsearch repository for 5.x packages
	baseurl=https://artifacts.elastic.co/packages/5.x/yum
	gpgcheck=1
	gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
	enabled=1
	autorefresh=1
	type=rpm-md

Save and exit.

Now install Elasticsearch:

    sudo yum -y install elasticsearch

Elasticsearch is now installed. Let's edit the configuration:

    sudo vim /etc/elasticsearch/elasticsearch.yml

You will want to restrict outside access to your Elasticsearch instance (port 9200), so outsiders can't read your data or shutdown your Elasticsearch cluster through the HTTP API. Find the line that specifies `network.host`, uncomment it, and replace its value with "localhost" so it looks like this:
elasticsearch.yml excerpt (updated)

    network.host: localhost

Save and exit elasticsearch.yml.

Now start & enable Elasticsearch to start automatically on boot up:

    sudo systemctl start elasticsearch
    sudo systemctl enable elasticsearch

## Install Kibana

The Kibana package shares the same GPG Key as Elasticsearch, and we already installed that public key.

Create and edit a new yum repository file for Kibana:

    sudo vim /etc/yum.repos.d/kibana.repo

Add the following repository configuration:
/etc/yum.repos.d/kibana.repo

	[kibana-5.x]
	name=Kibana repository for 5.x packages
	baseurl=https://artifacts.elastic.co/packages/5.x/yum
	gpgcheck=1
	gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
	enabled=1
	autorefresh=1
	type=rpm-md
	
Save and exit.

Install Kibana with this command:

    sudo yum -y install kibana

Open the Kibana configuration file for editing:

    sudo vim /opt/kibana/config/kibana.yml

In the Kibana configuration file, find the line that specifies `server.host`, and replace the IP address ("0.0.0.0" by default) with "localhost":

    server.host: "localhost"

Save and exit. This setting makes it so Kibana will only be accessible to the localhost. This is fine because we will have an Nginx reverse proxy, on the same server, to allow external access.

Now start the Kibana service, and enable it:

    sudo systemctl start kibana
    sudo systemctl enable kibana

Before we can use the Kibana web interface, we have to set up a reverse proxy. 

If you want to follow our setup, check how to do it here:

[Setting up a reverse proxy with Nginx.](nginx.md)

Otherwise, feel free to setup your own proxy so you can access the Kibana interface!


## Install Logstash

The Logstash package shares the same GPG Key as Elasticsearch, and we already installed that public key.

Create and edit a new yum repository file for Logstash:

    sudo vim /etc/yum.repos.d/logstash.repo

Add the following repository configuration:

	[logstash-5.x]
	name=Elastic repository for 5.x packages
	baseurl=https://artifacts.elastic.co/packages/5.x/yum
	gpgcheck=1
	gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
	enabled=1
	autorefresh=1
	type=rpm-md

Save and exit.

Install Logstash with this command:

    sudo yum -y install logstash

Logstash is installed but it is not configured yet.

**Note**: *We will configure it properly before setting it up and running!*
