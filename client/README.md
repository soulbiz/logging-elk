# Client Installation & Config

For our client, we just need Filebeat to send our log data to the Logstash server.
Also, we will need our client certificate to be validated by the same CA as our server for our setup.
This is better explained at the start of our server config guide.

## Installing Filebeat

Run the following command to import the Elasticsearch public GPG key into rpm:

    sudo rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch

Next, create and edit the Elastic repository:

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

Save and exit.

Now run the following command:

	sudo dnf -y install filebeat

OK! Once installed, we will edit the config file:

	sudo vim /etc/filebeat/filebeat.yml

First we have the **prospectors**.
Here we'll define which logs will be tailed and how will them be processed by Filebeat, and how 

