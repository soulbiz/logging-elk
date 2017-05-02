# Installation

We'll be covering the installation process for a CentOS 7 server.
Anyway, it should provide enough guidance to apply it in any other suitable distribution.

## Prerequisites

For our *ELK Server* (from now on, **ELKS**) we'll also need to install:

* Java 8 (As recommended by Elasticsearch ATM, or the latest version)
* Nginx (To setup a reverse proxy for external connections)

We will not cover that here, since we assume it should be easily done!

## Install Elasticsearch

Elasticsearch can be installed with a package manager by adding Elastic's package repository.

Run the following command to import the Elasticsearch public GPG key into rpm:

    sudo rpm --import http://packages.elastic.co/GPG-KEY-elasticsearch

Create a new yum repository file for Elasticsearch.

    sudo vim /etc/yum.repos.d/elasticsearch.repo

Add the following repository configuration:
/etc/yum.repos.d/elasticsearch.repo

    [elasticsearch-2.x]
    name=Elasticsearch repository for 2.x packages
    baseurl=http://packages.elastic.co/elasticsearch/2.x/centos
    gpgcheck=1
    gpgkey=http://packages.elastic.co/GPG-KEY-elasticsearch
    enabled=1

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

    [kibana-4.4]
    name=Kibana repository for 4.4.x packages
    baseurl=http://packages.elastic.co/kibana/4.4/centos
    gpgcheck=1
    gpgkey=http://packages.elastic.co/GPG-KEY-elasticsearch
    enabled=1

Save and exit.

Install Kibana with this command:

    sudo yum -y install kibana

Open the Kibana configuration file for editing:

    sudo vi /opt/kibana/config/kibana.yml

In the Kibana configuration file, find the line that specifies server.host, and replace the IP address ("0.0.0.0" by default) with "localhost":
kibana.yml excerpt (updated)

server.host: "localhost"

Save and exit. This setting makes it so Kibana will only be accessible to the localhost. This is fine because we will install an Nginx reverse proxy, on the same server, to allow external access.

Now start the Kibana service, and enable it:

    sudo systemctl start kibana
    sudo chkconfig kibana on

Before we can use the Kibana web interface, we have to set up a reverse proxy. Let's do that now, with Nginx.
Install Nginx

Because we configured Kibana to listen on localhost, we must set up a reverse proxy to allow external access to it. We will use Nginx for this purpose.

Note: If you already have an Nginx instance that you want to use, feel free to use that instead. Just make sure to configure Kibana so it is reachable by your Nginx server (you probably want to change the host value, in /opt/kibana/config/kibana.yml, to your Kibana server's private IP address). Also, it is recommended that you enable SSL/TLS.

Add the EPEL repository to yum:

    sudo yum -y install epel-release

Now use yum to install Nginx and httpd-tools:

    sudo yum -y install nginx httpd-tools

Use htpasswd to create an admin user, called "kibanaadmin" (you should use another name), that can access the Kibana web interface:

    sudo htpasswd -c /etc/nginx/htpasswd.users kibanaadmin

Enter a password at the prompt. Remember this login, as you will need it to access the Kibana web interface.

Now open the Nginx configuration file in your favorite editor. We will use vi:

    sudo vi /etc/nginx/nginx.conf

Find the default server block (starts with server {), the last configuration block in the file, and delete it. When you are done, the last two lines in the file should look like this:
nginx.conf excerpt

    include /etc/nginx/conf.d/*.conf;
}

Save and exit.

Now we will create an Nginx server block in a new file:

    sudo vi /etc/nginx/conf.d/kibana.conf

Paste the following code block into the file. Be sure to update the server_name to match your server's name:
/etc/nginx/conf.d/kibana.conf

    server {
        listen 80;

        server_name example.com;

        auth_basic "Restricted Access";
        auth_basic_user_file /etc/nginx/htpasswd.users;

        location / {
            proxy_pass http://localhost:5601;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection 'upgrade';
            proxy_set_header Host $host;
            proxy_cache_bypass $http_upgrade;        
        }
    }

Save and exit. This configures Nginx to direct your server's HTTP traffic to the Kibana application, which is listening on localhost:5601. Also, Nginx will use the htpasswd.users file, that we created earlier, and require basic authentication.

Now start and enable Nginx to put our changes into effect:

    sudo systemctl start nginx
    sudo systemctl enable nginx

Note: This tutorial assumes that SELinux is disabled. If this is not the case, you may need to run the following command for Kibana to work properly: sudo setsebool -P httpd_can_network_connect 1

Kibana is now accessible via your FQDN or the public IP address of your ELK Server i.e. http://elk\_server\_public\_ip/. If you go there in a web browser, after entering the "kibanaadmin" credentials, you should see a Kibana welcome page which will ask you to configure an index pattern. Let's get back to that later, after we install all of the other components.
Install Logstash

The Logstash package shares the same GPG Key as Elasticsearch, and we already installed that public key, so let's create and edit a new Yum repository file for Logstash:

    sudo vi /etc/yum.repos.d/logstash.repo

Add the following repository configuration:
/etc/yum.repos.d/logstash.repo

    [logstash-2.2]
    name=logstash repository for 2.2 packages
    baseurl=http://packages.elasticsearch.org/logstash/2.2/centos
    gpgcheck=1
    gpgkey=http://packages.elasticsearch.org/GPG-KEY-elasticsearch
    enabled=1

Save and exit.

Install Logstash with this command:

    sudo yum -y install logstash

Logstash is installed but it is not configured yet.
