---
name: FAQs
---

# FAQs

### Deployment

#### What do I do if I'm running another service on port 3000?

Change the listening port of Gogs the first time you run it with:

    ./gogs web -port 3001

This flag also changes the port number in the install page for you, so pick a number you want to assign for Gogs.

#### How do I use NGINX with Reverse Proxy?

Add following `server` section inside the `http` section of `nginx.conf` and reload the configuration:

```
server {
    listen 80;
    server_name git.crystalnetwork.us;

    location / {
        proxy_pass http://localhost:3000;
    }
}
```

##### How do I set up a sub-URL with NGINX?

In case you need to use a sub-path for your Gogs instance, you can change your NGINX configuration to the following
(note to the suffix `/`):

```
server {
    listen 80;
    server_name git.crystalnetwork.us;

    location /gogs/ {
        proxy_pass http://localhost:3000/;
    }
}
```

Then set `[server] ROOT_URL = http://git.crystalnetwork.us/gogs/` in your configuration.

##### Why am I getting errors when uploading large files?

To allow NGINX to handle large file uploads in repositories, please see a relevant discussion [here](http://stackoverflow.com/a/15021750). `413` is a common NGINX error; append following line to your server block to fix this:

```
client_max_body_size 50m;
```

##### How do I set up a sub-URL with Apache 2?

Use following configuration template:

```
<VirtualHost *:443>
        ...
        <Proxy *>
                 Order allow,deny
                 Allow from all
        </Proxy>

        ProxyPass /git http://127.0.0.1:6000
        ProxyPassReverse /git http://127.0.0.1:6000
</VirtualHost>
```

##### How do I set up a sub-URL with lighttpd?

Use following configuration template:

```
server.modules  += ( "mod_proxy_backend_http" )
$HTTP["url"] =~ "^/gogs" {
        proxy-core.protocol = "http"
        proxy-core.backends = ( "localhost:3000" )
        proxy-core.rewrite-request = (
          "_uri" => ( "^/gogs/?(.*)" => "/$1" ),
          "Host" => ( ".*" => "localhost:3000" ),
        )
}
```

#### How do I set up HTTPS?

Change following configuration options in `custom/conf/app.ini` in the section that looks like this sample:

```
[server]
PROTOCOL = https
ROOT_URL = https://try.gogs.io/
CERT_FILE = custom/https/unified.cert
KEY_FILE = custom/https/decryped-private.key
```

If you want to use self-signed HTTPS, you can execute the following command to generate a certificate and key files:

	$ ./gogs cert -ca=true -duration=8760h0m0s -host=myhost.example.com

#### How do I run Gogs in offline mode/in an intranet?

To run Gogs in an intranet, change the configuration option `server -> OFFLINE_MODE` to `true` in the file `custom/conf/app.ini`.

#### How do I make a custom robots.txt?

Create a file called `robots.txt` in the `custom` directory.

#### How do I run Gogs as a daemon?

Gogs has some third-party scripts that support running it as a daemon:

- [init.d/centos](https://github.com/gogits/gogs/blob/master/scripts/init/centos/gogs)
- [init.d/debian](https://github.com/gogits/gogs/blob/master/scripts/init/debian/gogs)
- Systemd in the following section.

#### How do I run Gogs at startup with Systemd?

Three's a [systemd service template file](https://github.com/gogits/gogs/blob/master/scripts/systemd/gogs.service) in the Gogs GitHub repository. It needs some modifications for a working version for your installation:

1. Replace the `start.sh` path of `ExecStart` with the path of your Gogs installation.
2. Also replace the path of `WorkingDirectory` with the path of your Gogs installation.
3. (Optional) If you are would like to use Gogs with `MySQL/MariaDB`, `PostgreSQL`, `Redis`, or `memcached`, uncomment the line corresponding to `After`.

When you are complete with your modification of the systemd file, save it in `/etc/systemd` and start it with `sudo systemd restart gogs`.

You can check the status of the Gogs systemd service with `sudo systemd status gogs -l` or display the journald entries directly with `sudo journalctl -b -u gogs.service`.

### Administration

#### How can I become an administrator?

The first registered user with `ID = 1` is an administrator. No e-mail confirmation is required for this (if enabled). The default administrator can log into `Admin` > `Users` and authorize another user. A user will also be an administrator if they register in the install page.

### Repository Management

#### How do I give Git Hooks permission to users?

This is a **high-level permission which can damage your system** which you must enable/disable in the admin user management panel (`/admin/users/:userid`). Only grant this permission to users who you really trust.

### Other

#### How do I find the current version of Gogs?

The current version of Gogs is written in plain text in the file `templates/.VERSION`.

#### What is the `gogs serv` command for?

You have no reason to execute this command manually; it will be called by the Git update hook whenever new SSH pushes arrive.
