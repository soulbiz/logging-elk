# Configuration

The scope of this document is setting up our ELK Server once gone through the installation process and initial setup tweaks.
If you skipped our [Installation Guide](../installation/), feel free to take a quick look at it!

## Creating our own CA and SSL Certificates

Because Filebeat will be sending all of our logs, we do not want them to be visible for untrusted hosts as they travel.
You can use SSL mutual authentication to secure connections between Filebeat and Logstash. This ensures that Filebeat sends encrypted data to trusted Logstash servers only, and that the Logstash server receives data from trusted Filebeat clients only.

We'll be using the following infrastructure:

* Server certificate
* Client certificates
* A main CA to validate both server and client certificates.

We have to make sure that our certificates are signed properly by our CA. Otherwise, we'll be unable to stablish a connection!
That way we can make sure that no information is sent to unwanted servers or from unwanted clients.

We will not cover the process to create a correct infrastructure here.
Check this Elastic reference for more information on the SSL config between Filebeat and Logstash:

[Securing Communication With Logstash by Using SSL](https://www.elastic.co/guide/en/beats/filebeat/current/configuring-ssl-logstash.html)

## Configure Logstash

Logstash configuration files are in the JSON-format, and reside in /etc/logstash/conf.d.
The configuration basically consists of three sections:

* [Inputs](#logstash-inputs)
* [Filters](#logstash-filters)
* [Outputs](#logstash-outputs)

### Logstash Inputs

Let's create a configuration file called `02-beats-input.conf` and set up our "filebeat" input:

    sudo vim /etc/logstash/conf.d/02-beats-input.conf

Insert the following input configuration:

	02-beats-input.conf

    input {
      beats {
        port => 5044
        ssl => true
        ssl_certificate_authorities => ["/etc/pki/tls/certs/my-ca.crt"]
        ssl_certificate => "/etc/pki/tls/certs/elk-server.crt"
        ssl_key => "/etc/pki/tls/private/elk-server.key"
        ssl_verify_mode => "force_peer"
      }
    }

Save and quit. This specifies a beats input that will listen on tcp port 5044, and it will use the SSL certificate and private key that we created for our server.
We'll cover the filebeat client config later on. Be sure to change the certificates and key paths with your own!

### Logstash Filters

For our setup, we'll be filtering this log sources:

* Audit
* Samba
* Radius

You can access our config files for this purpose here:

[Logstash Config Filters](files/)

Those filters look for logs that are labeled as specified on the "document_type" option on Filebeat config. They will try to use Grok to parse incoming logs to make it structured and query-able.

You can check a more specific explanation of our configuration here:

[Grok Filters & Patterns](grok-patterns-draft.md)

### Logstash Output

Lastly, we will create a configuration file called `30-elasticsearch-output.conf`:

    sudo vim /etc/logstash/conf.d/30-elasticsearch-output.conf

Insert the following output configuration:

	/etc/logstash/conf.d/30-elasticsearch-output.conf

    output {
      elasticsearch {
        hosts => ["localhost:9200"]
        sniffing => true
        manage_template => false
        index => "%{[@metadata][beat]}-%{+YYYY.MM.dd}"
        document_type => "%{[@metadata][type]}"
      }
    }

Save and exit. This output basically configures Logstash to store the beats data in Elasticsearch which is running at `localhost:9200`, in an index named after the beat used (filebeat, in our case). We could also configure other outputs elsewhere, for example to a file.

If you want to add filters for other applications that use the Filebeat input, be sure to name the files so they sort between the input and the output configuration (i.e. between 02- and 30-).

Start and enable Logstash to put our configuration into effect:

    sudo systemctl start logstash
    sudo systemctl enable logstash

## Starting with Kibana

Now that we have our server up and running (and if you followed our steps), we should be able to visualize something!

Head to the server Kibana interface at `http://elk_server_public_ip/`.
If you configured Nginx as we recommended in our Installation Guide, after logging in with our credentials, a page to config the default index pattern should appear.

Go ahead and select the `filebeat` index pattern (that we previously loaded) from the Index Patterns menu (left side), then click the Star (Set as default index) button to set the Filebeat index as the default.

Once we've done that, we can head to "Discover" at the top navigation bar. Here we can see all log data from Elasticsearch (in a more fancy way).
You should be able to see a histogram + the full log messages below.

If you click on an event, it should display the parsed fields that we filtered with Grok in the Logstash filters.
We can see the log data in a table-like display or the full JSON document as it is stored in Elasticsearch (with the new fields added by our filters!).





