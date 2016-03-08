#GoPhish Docker Container
##Running
The suggested way of running the container is to use Docker Compose. The docker-compose.yml uses a config.json file to enable HTTPS.

__Download the docker-compose.yml and config.json files:__
>git clone https://github.com/scottg88/gophish-compose.git

Edit the docker-compose.yml file for any environment-specific settings. E.g. update the "volumes" section if you do not like the default locations.

Copy the "admin" and "phish" certificates to /srv/gophish/ (by default) and __update the permissions:__
>chmod 0600 /srv/gophish/admin.* /srv/gophish/phish.*

__Start the container:__
>docker-compose up -d

Alternatively run the container directly (by default using HTTP and persisting the database to a unnamed volume):
>docker run -d --name gophish -p 3333:3333 -p 80:80 --restart=always scottg88/gophish

##Upgrading
Upgrading without Docker Compose and default _docker run_ settings from above:

>docker stop _gophish_
>docker rename _gophish_ _gophish-old_
>docker run -d --name _gophish_ -p 3333:3333 -p 80:80 --restart=always --volumes-from=_gophish-old_ scottg88/gophish
>docker rm _gophish-old_

Italics indicate the container names, ensure they match names used in your environment.

Upgrading with Docker Compose, perform the following from the folder holding docker-compose.yml:

>docker-compose stop

>docker-compose down

>docker-compose pull

>docker-compose up -d

##Persisting Data
###Database
The configuration file (config.json) is modified to place the database file (gophish.db) in a sub-directory. This allows a volume to be defined to persist the gophish database file.
>__Config snippet:__
>"db_path" : "database/gophish.db",

A volume is defined in the Dockerfile persisting the database by default. Alternatively mount a volume against "_/app/database_" with _docker run_.

##HTTPS
To use HTTPS with this container, overwrite the config file with a volume mount at _docker run_.

__Config file:__ /app/config.json

__Example config snippet:__
>        "admin_server" : {
>                "listen_url" : "0.0.0.0:3333",
>                "use_tls" : true,
>                "cert_path" : "admin.crt",
>                "key_path" : "admin.key"
>        },
>        "phish_server" : {
>                "listen_url" : "0.0.0.0:443",
>                "use_tls" : true,
>                "cert_path" : "phish.crt",
>                "key_path": "phish.key"
>        },

If using a config.json with the above settings, mount the following files at _docker run_:

- /app/admin.crt (admin interface pub key)
- /app/admin.key (admin interface priv key)
- /app/phish.crt (phishing interface pub key)
- /app/phish.key (phishing interface priv key)

>__Example__:
>docker run -d --name gophish --restart=always -p 3333:3333 -p 443:443 -v /path/to/config.json:/app/config.json -v /path/to/admin.crt:/app/admin.crt -v /path/to/admin.key:/app/admin.key -v /path/to/phish.crt:/app/phish.crt -v /path/to/phish.key:/app/phish.key scottg88/gophish



