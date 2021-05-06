# statuspage
Status Page Editan
This is the official repository of the [Docker image](https://hub.docker.com/r/cachethq/docker/) for [Cachet](https://github.com/CachetHQ/Cachet).

[Cachet](https://github.com/CachetHQ/Cachet) is a beautiful and powerful open source status page system, a free replacement for services such as StatusPage.io, Status.io and others.

For full documentation, visit the [Installing Cachet with Docker](https://docs.cachethq.io/docs/get-started-with-docker) page.

# Supporting Cachet

Cachet is a BSD-3-licensed open source project. If you'd like to support future development, check out the [Cachet Patreon](https://patreon.com/jbrooksuk) campaign.

# Just For Internal Itsnan Project

# Quickstart

1. Clone this repository

  ```shell
  git clone https://github.com/akangitsnan/statuspage.git
  ```

2. Edit the docker-compose.yml file to specify your [ENV variables](/conf/.env.docker).


3. To build an image containing a specific Cachet release, set the [`cachet_ver` ARG in the docker-compose.yml](/docker-compose.yml)

  The *main* branch and *cachethq/docker:latest* Docker automated build are a work in progress / development version of the upstream https://github.com/CachetHQ/Cachet project. As such, *main* or *latest* should not be used in a production environment as it can change at anytime.

  We strongly recommend specifying a stable [Cachet Release](https://github.com/CachetHQ/Cachet/releases) at build time as mentioned above.

4. Build and run the image

  ```shell
  docker-compose build
  docker-compose up
  ```

5. `cachethq/docker`  runs on port 8000 by default. This is exposed on host port 80 when using docker-compose.


6. Setup the APP_KEY

Whilst the container is up and running, find the name of the Cachet container via `docker ps`.

Run `docker exec -i ID_OF_THE_CONTAINER php artisan key:generate`.

Replace `${APP_KEY:-null}` in `docker-compose.yml` with the newly generated Application key.

__Note:__ make sure you include `base64:` prefix. E.g. `base64:YOUR_UNIQUE_KEY`

Restart the Docker containers.


# Docker Hub Automated build

`cachethq/docker` is available as a [Docker Hub Trusted Build](https://hub.docker.com/r/cachethq/docker/).

For a full list of Cachet versions released as Docker images  please see the [list of Docker hub tags](https://hub.docker.com/r/cachethq/docker/tags/).

Please use a [tagged Cachet Docker image release](https://github.com/CachetHQ/Docker/releases) or one of the tagged builds from https://hub.docker.com/r/cachethq/docker/tags/ with `docker pull cachethq/docker:2.3.12`.

# Debugging

* The services such as Cachet, supervisord, nginx, and php-fpm log to `stdout` and `stderr`, and are visible in the Docker runtime output. 

* Setting the `DEBUG` Docker environment variable within the `docker-compose.yml` file or at runtime to `true` will enable debugging of the container entrypoint init script.

# Testing

Pull requests must pass the [Bash Automated Testing System](https://github.com/sstephenson/bats) tests, which run on [Travis CI](https://travis-ci.org/CachetHQ/Docker) via located in the [test](test) directory.

Use `make test` to manually run the tests.


# Development of Cachet using this docker environment

1.  Clone the official repo of CachetHQ/Docker:

  ```shell
  git clone https://github.com/CachetHQ/Docker.git cachet-docker
  cd cachet-docker
  git tag -l
  git checkout $LATEST_TAG
  ```
2. Clone the official repo of CachetHQ/Cachet here and do composer install:

  ```shell
  git clone https://github.com/CachetHQ/Cachet.git
  ```

3. Setup the Cachet project:

Note: This requires [Composer](https://getcomposer.org/) be installed on your Docker host.  

 ```shell
cd Cachet
composer install
cp ../conf/.env.docker ./.env
cd ..
```

4. Edit the docker-compose.yml file to bind mount the repo as a volume:

  ```yaml
 cachet:
    volumes:
      - ./Cachet/:/var/www/html/
    ...  
  ```

5. Build and run the container:

  ```shell
  docker-compose up
  ```

6. Open new terminal and run the following commands after getting container name via `docker ps`:

  ```shell
  docker exec -i cachetdocker_cachet_1  php artisan key:generate
  docker exec -i cachetdocker_cachet_1  php artisan app:install
  ```
7. Add SSL Certificate to cacher docker container, create default.conf on host docker :

 ```shell
 server {
    # This port is mapped to port 80 on the host
    listen 8000;

    # Your domain name
    server_name your_domain_name.com;

    # Redirects http requests to https
    return 301 https://$server_name$request_uri;
}
server {
    # This port is mapped to port 443 on the host
    listen 8003 default; ## Listen for ipv4; this line is default and implied

    ssl on;
    ssl_certificate /etc/nginx/ssl/your_domain_name.crt;
    ssl_certificate_key /etc/nginx/ssl/your_domain_name.key;


    # Make site accessible from http://localhost/
    server_name your_domain_name.com;
    root /var/www/html/public;

    index index.html index.htm index.php;

    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    # Pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    location ~ \.php$ {
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        include fastcgi_params;
        fastcgi_pass 127.0.0.1:9000;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_index index.php;
        fastcgi_keep_conn on;
    }

    location ~ /\.ht {
        deny all;
    }

}

 ```

8. Use the docker cp command to copy your SSL configuration file to the Cachet containers nginx configuration directory :

 ```shell
 docker cp default.conf {id_container/container_name}:/etc/nginx/conf.d/default.conf
 ```

9. Copy your certificate (your_domain_name.crt) and key (your_domain_name.key) files to the /etc/nginx/ directory.

 ```shell
 docker cp your_domain_name.crt cachet:/etc/nginx/your_domain_name.crt
 docker cp your_domain_name.key cachet:/etc/nginx/your_domain_name.key
 ```
