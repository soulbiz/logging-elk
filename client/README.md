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

## Configuring Filebeat

OK! Once installed, we will edit the config file:

	sudo vim /etc/filebeat/filebeat.yml

First we have the **prospectors**.
Here we'll define which logs will be tailed and how will them be processed by Filebeat.

Each `-` is a prospector. We can config different options:

* `input_type`: It defines that it will read the input line by line
* `paths`: Those will be the directories or files to be used. We can use regexp to choose them!
* `document_type`: It will define in wich "type" will those documents be indexed by Elastic by adding a field with that value. **Watch out!** We need that value to match the "type" value in our server's Logstash Grok Filters so we apply the appropiate filters. 
* `multiline`: Those 3 options will define which lines are the initials of a single log-line that span multiple lines of text. We can specify which pattern they should match to be considered as a part of a single multiline event. *Chech the Filebeat references for more information about multiline matching*.

We will be using the following input config for our clients:


	#=========================== Filebeat prospectors =============================

	- input_type: log
	  paths:
		- /var/tmp/logs/radius/radius*.log
	  document_type: radius

	- input_type: log
	  paths:
		- /var/tmp/logs/radius/detail-*
	  document_type: radius-detail

	  multiline.pattern: '^[[:space:]]'
	  multiline.negate: false
	  multiline.match: after

	- input_type: log
	  paths:
		- /var/tmp/logs/samba/*.log
	  document_type: samba
	  
	  multiline.pattern: '^\[[0-9]{4}\/[0-9]{2}\/[0-9]{2}'
	  multiline.negate: true
	  multiline.match: after

	- input_type: log

	  paths:
		- /var/log/audit/*.log
		#- c:\programdata\elasticsearch\logs\*

	  document_type: audit


With that, we'll be sending our main radius logs, radius-session detail logs, samba logs and audit logs.
**Remember to change the "paths" values to match your log files!!**

Now for our output, we want to send the processed events to Logstash to be filtered before ending up into Elasticsearch.
Find the `output.elasticsearch` block and erase it or comment all its lines, as we are not going to use it.

Our Logstash output for this setup should look like that:

	#----------------------------- Logstash output --------------------------------
	output.logstash:
	  # The Logstash hosts
	  hosts: ["elk_server:5044"]

	  # Optional SSL. By default is off.
	  # List of root certificates for HTTPS server verifications
	  ssl.certificate_authorities: ["/etc/pki/tls/certs/ca_edt.crt"]

	  # Certificate for SSL client authentication
	  ssl.certificate: "/var/tmp/certificates/omar_client_2.crt"

	  # Client Certificate Key
	  ssl.key: "/var/tmp/certificates/omar_client_2.key"


As we can see, we need to specify the Logstash's host where we'll send our events. Change the `elk_server` hostname with your own! We can specify more than one if we want in its list value. 

For the SSL config, we are using a both server and client verification. As stated in our [server configuration guide](../config), we will need our main CA certificate to verify the one from the server. Also, to be able to send data there, we need our own key and certificate that should be validated by the same CA. Be sure to change those values with your certificates and key paths!

We can also specify a straight file output which we can use to see how our data is being sent!

	#----------------------------- File output --------------------------------
	output.file:
		path: /tmp/filebeat
		filename: filebeat

Now we can start our Filebeat client:

    sudo systemctl start filebeat
    sudo systemctl enable filebeat

With that, we should have our client ready and sending log events to our server.

## References

* [**Elastic** - Filebeat Reference](https://www.elastic.co/guide/en/beats/filebeat/current/index.html)
* [**Digital Ocean - Mitchell Anicas**: How To Install ELK on CentOS 7 - Adding Client Servers](https://www.digitalocean.com/community/tutorials/how-to-install-elasticsearch-logstash-and-kibana-elk-stack-on-centos-7#set-up-filebeat-(add-client-servers))

