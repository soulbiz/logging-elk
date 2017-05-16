# Configuration

## Creating our own CA and SSL Certificates

We'll be using the following infrastructure:

* Server certificate
* Client certificates
* A main CA to validate both server and client certificates.

Using that we'll be able to ensure a secure communication on both ends.
We have to make sure that our certificates are signed properly by our CA. Otherwise, we'll be unable to stablish a connection!
That way we can make sure that no information is sent to unwanted servers or from unwanted clients.

Creating a correct SSL/TLS infrastructure is outside the scope of this document. There are many online resources available for this purpose.

## Configure Logstash

Logstash configuration files are in the JSON-format, and reside in /etc/logstash/conf.d.
The configuration consists of three sections:

* [Inputs](#logstash-inputs)
* [Filters](#logstash-filters)
* [Outputs](#logstash-outputs)

### Logstash Inputs

Let's create a configuration file called 02-beats-input.conf and set up our "filebeat" input:

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
Be sure to change the certificates and key paths with your own!

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

Lastly, we will create a configuration file called 30-elasticsearch-output.conf:

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

Save and exit. This output basically configures Logstash to store the beats data in Elasticsearch which is running at `localhost:9200`, in an index named after the beat used (filebeat, in our case).

If you want to add filters for other applications that use the Filebeat input, be sure to name the files so they sort between the input and the output configuration (i.e. between 02- and 30-).

Test your Logstash configuration with this command:

    sudo service logstash configtest

It should display Configuration OK if there are no syntax errors. Otherwise, try and read the error output to see what's wrong with your Logstash configuration.

Restart and enable Logstash to put our configuration changes into effect:

    sudo systemctl start logstash
    sudo systemctl enable logstash

Next, we'll load the sample Kibana dashboards.
