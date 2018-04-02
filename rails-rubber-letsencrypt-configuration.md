## Let's Encrypt in Rails with Capistrano and Rubber

### Haproxy config
* `config/rubber/role/haproxy/haproxy-passenger.conf`
	
	Make sure that `mode` is set to `tcp`

### First Steps Nginx Confifguration

* `config/rubber/role/passenger_nginx/passenger_nginx.conf`
* `config/rubber/role/passenger_nginx/nginx-tools.conf`

In the section below `<% if rubber_env.use_ssl_key %>` add the following, (the ssl\_protocols and ssl\_ciphers should always be updated to the most secure and up to date order possible.)
 
```
server {
	ssl on;
	listen <%= rubber_env.passenger_listen_ssl_port %>;

	proxy_set_header X_FORWARDED_PROTO https;

    
	# SSL certificate and key
	ssl_certificate  /etc/letsencrypt/live/<%= #{rubber_env.app_name}.#{rubber_env.domain} %>/fullchain.pem;
	ssl_certificate_key  /etc/letsencrypt/live/<%= #{rubber_env.app_name}.#{rubber_env.domain %>/privkey.pem;

	# SSL configuration
	ssl_session_timeout 1d;
	ssl_session_cache shared:SSL:50m;
	ssl_session_tickets off;

	# Diffie-Hellman parameter for DHE ciphersuites, recommended 2048 bits
	ssl_dhparam /etc/ssl/certs/dhparam.pem;

	# intermediate configuration. tweak to your needs.
	ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
	ssl_ciphers 'ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA:ECDHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA256:DHE-RSA-AES256-SHA:ECDHE-ECDSA-DES-CBC3-SHA:ECDHE-RSA-DES-CBC3-SHA:EDH-RSA-DES-CBC3-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:DES-CBC3-SHA:!DSS';
	ssl_prefer_server_ciphers on;
	
	...
}
```
Using the app name and domain combination for the domain name is theconvention, but if you register a differenct URL then the app name you will have to cusomize the certificate path.

Check [Mozilla SSL Configuration Generator](https://mozilla.github.io/server-side-tls/ssl-config-generator/) for most up to date cipher configuration for the server's version of Nginx.

The first deployment you will need to ensure https is off to allow connections through http.
After deployment change `use_ssl_key` to `true` in `config/rubber/role/rubber-passenger_nginx.yml`

Do the same in `config/rubber/role/web_tools/nginx-tools.conf` if you have the role 'web tools'. (It will need to be added in two places.)

Add to the second server block in `config/rubber/role/passenger_nginx/passenger_nginx.conf`

``` 
location ^~ /.well-known/ {
    allow all;
}
``` 

Add folder named `/.well-known/` to the `public` in the root application directory.

```
$ mkdir public/.well_known
```

### Installing Let’s Encrypt
Use SSH to log into your server as root user. 

### Configuring Let's Encrypt

First download and install [Certbot](https://certbot.eff.org/), [14.04](https://certbot.eff.org/lets-encrypt/ubuntutrusty-nginx),
[12.04](https://certbot.eff.org/lets-encrypt/ubuntuother-nginx)

#### Ubuntu > 14.04 and Nginx
```
$ sudo apt-get update
$ sudo apt-get install software-properties-common
$ sudo add-apt-repository ppa:certbot/certbot
$ sudo apt-get update
$ sudo apt-get install python-certbot-nginx 
```
#### Ubuntu < 14.04 and Nginx

Ubuntu versions below 14.04 do not have a packaged version of certbot. We have to use the certbot-auto script.

```
$ sudo mkdir /etc/letsencrypt
$ cd /etc/letsencrypt
$ sudo wget https://dl.eff.org/certbot-auto
$ sudo chmod a+x certbot-auto
```

#### Configuration
The configuration needed to create the certificates are put in a file named `cli.ini` inside `/etc/letsencrypt/`,

```
rsa-key-size = 4096
email = <your-email>
domains = <domains>
authenticator = webroot
webroot-path = /mnt/<app_name>/current/public
```

You need to provide your email address for recovering the certificate credentials and for if a certificate renewal should fail. (Diff: use systemintegrations@diffagency.com) Also add the domains for which you want the certificates for seperated by commas like, `example.com, www.example.com`.

### Creating Certificate
Finally, we are ready to create our first certificate. Execute the following commands,

```
# Ubuntu > 14.04
$ sudo certbot certonly
# Ubuntu < 14.0.4
$ sudo certbot-auto certonly
```

This creates the SSL certificates in `/etc/letsencrypt/live/<domain-name>/` folder. Whenever we renew the certificates, the latest will be found in this folder.

### Automating Certificate Renewal
Let’s Encrypt certificates are valid for 90 days, so we need to renew them. To renew, you just have to run the client with `renew` command,

```
# Ubuntu > 14.04
$ sudo certbot certonly
# Ubuntu < 14.0.4
$ sudo certbot-auto certonly
```

This command will renew the certificate. We can automate renewal by running this command as a cron job. We can make this command run once a month to renew certificates at a monthly basis. We also need to reload the nginx configurations.

To add a new cron job, type the following command:

`sudo crontab -e`

Add the following lines to the end of the cron file:

```
SHELL=/bin/bash
HOME=/
MAILTO=”example@mail.com”

...
# Ubuntu > 14.04
0 4 * * * (certbot renew --post-hook "service nginx reload") >> /var/log/letsencrypt.log

# Ubuntu < 14.04
0 4 * * * (/etc/letsencrypt/certbot-auto renew --post-hook "service nginx reload") >> /var/log/letsencrypt.log
```

This will cause the command to run at 4:00AM everyday. The cert will only be renewed if there is less then 30 days remain The command to restart Nginx in the `--post_hook` flag will  only run when the cert is renewed. Command The output of this command is stored in `/var/log/letsencrypt.log`.

### Adding Forward Secrecy & Diffie Hellman Ephemeral Parameters
We need generate a DHE parameter:

```
$ cd /etc/ssl/certs
$ openssl dhparam -out dhparam.pem 4096
```

### Activating HTTPS in the App
Now that the app has and the server are set up for HTTPS. It is time to turn it on. 
Simply change the line in `config/rubber/rubber-passenger_nginx.yml`:
`use_ssl_key: true`

Then redeploy the configurtion: `cap deploy`





