# Installation

We'll be covering the installation process for a Fedora 23 server.
Anyway, it should provide enough guidance to apply it in any other suitable distribution.

## Prerequisites

For our *ELK Server* (from now on, **ELKS**) we'll also need to install:

* Java 8 (As recommended by Elasticsearch, Java 9 not supported)
* Nginx (To setup a reverse proxy for external connections)

Our ELK Stack can be installed with a package manager by adding Elastic's package repository.
Run the following command to import the Elasticsearch public GPG key into rpm:

    sudo rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch

Next, create and edit the Elastic repository for us to use:

	sudo vim /etc/yum.repos.d/elasticsearch.repo

Add the following lines:

	[elastic-5.x]
	name=Elastic repository for 5.x packages
	baseurl=https://artifacts.elastic.co/packages/5.x/yum
	gpgcheck=1
	gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
	enabled=1
	autorefresh=1
	type=rpm-md

With that we'll install the latest 5.x version available for all of our Elastic services.

## Install Elasticsearch

Run the following command to install Elasticsearch:

    sudo dnf -y install elasticsearch

Elasticsearch is now installed. Let's edit the configuration:

    sudo vim /etc/elasticsearch/elasticsearch.yml

You will want to restrict outside access to your Elasticsearch instance (port 9200), so outsiders can't read your data or shutdown your Elasticsearch cluster through the HTTP API. Find the line that specifies `network.host`, uncomment it, and replace its value with `localhost` so it looks like this:

elasticsearch.yml excerpt (once updated)

    network.host: localhost

Save and exit elasticsearch.yml.

Now start & enable Elasticsearch to start automatically on boot up:

    sudo systemctl start elasticsearch
    sudo systemctl enable elasticsearch

## Install Kibana

Install Kibana with this command:

    sudo dnf -y install kibana

By default, Kibana only listens to `localhost`.

This is fine because we will have an Nginx reverse proxy, on the same server, to allow external access.

Now start the Kibana service, and enable it:

    sudo systemctl start kibana
    sudo systemctl enable kibana

Before we can use the Kibana web interface, we have to set up a reverse proxy. 

If you want to follow our setup, check how to do it here:

[Setting up a reverse proxy with Nginx.](nginx.md)

Otherwise, feel free to setup your own proxy so you can access the Kibana interface!

## Install Filebeat

We'll be installing the Filebeat package so we can extract and load the Elasticsearch Index Template and the Kibana Index Pattern.
The recommended index template file for Filebeat is installed by the Filebeat packages.

Run the following command:

	sudo dnf -y install filebeat
	
Now, we DON'T need to start it!
We'll just use what we need from it.

Because we need to send the output to Logstash to parse our logs,
we need to manually load the Filebeat template first into Elasticsearch.

In Elasticsearch, index templates are used to define settings and mappings that determine how fields should be analyzed.
We need to do that from `localhost`, as we blocked outside access for security reasons.

Just run the following command:

	curl -H 'Content-Type: application/json' -XPUT 'http://localhost:9200/_template/filebeat' -d@/etc/filebeat/filebeat.template.json

Now that we have that, we need to load the Index Pattern.
We can do that with the help of their own script:

	/usr/share/filebeat/scripts/import_dashboards -only-index

Once we’ve loaded the index pattern, we can select the `filebeat-*` index pattern in Kibana to explore Filebeat data.
But first we have to finish with our setup!

## Install Logstash

Install Logstash with this command:

    sudo dnf -y install logstash

Logstash is installed but it is not configured yet.

**Note**: *We will configure it properly before setting it up and running!*

From here we'll start tuning our Logstash service to parse the log types that we want to visualize on Kibana.

Let's keep going!

[ELK Configuration](../config/)

## References:

[**Digital Ocean - Mitchell Anicas**: How To Install ELK on CentOS 7](https://www.digitalocean.com/community/tutorials/how-to-install-elasticsearch-logstash-and-kibana-elk-stack-on-centos-7)
