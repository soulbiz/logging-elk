# Reverse Proxy w/ Nginx

Because we configured Kibana to listen on `localhost`, we must set up a reverse proxy to allow external access to it. We will use Nginx for this purpose.

Use yum to install Nginx and httpd-tools:

    sudo dnf -y install nginx httpd-tools

Use htpasswd to create an admin user, called "kibadmin" (or better: use any other name!), that can access the Kibana web interface:

    sudo htpasswd -c /etc/nginx/htpasswd.users kibadmin

Enter a password at the prompt. We will use that to access the Kibana web interface.

Now open the Nginx configuration file:

    sudo vim /etc/nginx/nginx.conf

Find the default server block (starts with `server {`), the last configuration block in the file, and delete it. When you are done, the last two lines in the file should look like this:

    include /etc/nginx/conf.d/*.conf;
    }

Save and exit.

Now we will create an Nginx server block in a new file:

    sudo vim /etc/nginx/conf.d/kibana.conf

Paste the following code block into the file. Be sure to update the `server_name` to match your server's name or IP:

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

Save and exit. This configures Nginx to direct your server's HTTP traffic to the Kibana application, which is listening on `localhost:5601`. Also, Nginx will use the `htpasswd.users` file, that we created earlier, and require basic authentication.

Now start and enable Nginx to put our changes into effect:

    sudo systemctl start nginx
    sudo systemctl enable nginx

**Note**: *This tutorial assumes that SELinux is disabled.*

Kibana is now accessible via your FQDN or the public IP address of your **ELKS** i.e. `http://elk\_server\_public\_ip/`.
