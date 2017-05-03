# Configuration

## Creating our own CA and SSL Certificates

## Configure Logstash

Logstash configuration files are in the JSON-format, and reside in /etc/logstash/conf.d. The configuration consists of three sections: inputs, filters, and outputs.

Let's create a configuration file called 02-beats-input.conf and set up our "filebeat" input:

    sudo vim /etc/logstash/conf.d/02-beats-input.conf

Insert the following input configuration:
02-beats-input.conf

    input {
      beats {
        port => 5044
        ssl => true
        ssl_certificate => "/etc/pki/tls/certs/logstash-forwarder.crt"
        ssl_key => "/etc/pki/tls/private/logstash-forwarder.key"
      }
    }

Save and quit. This specifies a beats input that will listen on tcp port 5044, and it will use the SSL certificate and private key that we created earlier.

Now let's create a configuration file called 10-syslog-filter.conf, where we will add a filter for syslog messages:

    sudo vim /etc/logstash/conf.d/10-syslog-filter.conf

Insert the following syslog filter configuration:
10-syslog-filter.conf

    filter {
      if [type] == "syslog" {
        grok {
          match => { "message" => "%{SYSLOGTIMESTAMP:syslog_timestamp} %{SYSLOGHOST:syslog_hostname} %{DATA:syslog_program}(?:\[%{POSINT:syslog_pid}\])?: %{GREEDYDATA:syslog_message}" }
          add_field => [ "received_at", "%{@timestamp}" ]
          add_field => [ "received_from", "%{host}" ]
        }
        syslog_pri { }
        date {
          match => [ "syslog_timestamp", "MMM  d HH:mm:ss", "MMM dd HH:mm:ss" ]
        }
      }
    }

Save and quit. This filter looks for logs that are labeled as "syslog" type (by Filebeat), and it will try to use grok to parse incoming syslog logs to make it structured and query-able.

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

    sudo systemctl restart logstash
    sudo chkconfig logstash on

Next, we'll load the sample Kibana dashboards.
