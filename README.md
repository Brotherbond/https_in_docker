# HTTPS in docker 

install mkcert on ur pc

##### If it's the firt install of mkcert, run
mkcert -install

##### Generate certificate for domains of interest "docker.localhost", "domain.local" and their sub-domains
mkcert -cert-file ./proxy/certs/_wildcard.pem -key-file ./proxy/certs/_wildcard-key.pem "docker.localhost" "*.docker.localhost" "domain.local" "*.domain.local"

NB:just one level with the '*' such as "*.test" or "*.localhost" will be flagged as insecure



As a web application developer, one of the most common challenge faced is, not having the local development environment close enough to the production environment. While there can be many aspects to this, in this post we will focus on the following two

- Having a domain name, instead of something like `http://localhost:8080`
- Having a valid `HTTPS` certificate on the local development machine

Also, I am going to use docker for this demonstration. So lets begin.

I am going to use Wordpress as my example application. So lets head over the the docker page for [Wordpress](https://hub.docker.com/_/wordpress). Scrolling to the bottom shows us a sample configuration which can be used with docker-compose to have Wordpress running with [MySQL](https://hub.docker.com/_/mysql).

Open a terminal and make a folder with name, say `wordpress-with-http` and move inside it. Now create a file with name `docker-compose.yml` and paste the contents copied from [here](https://hub.docker.com/_/wordpress). 

```
version: '3.9'

services:
  wordpress:
    image: wordpress
    restart: always
    ports:
      - 8080:80
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: exampleuser
      WORDPRESS_DB_PASSWORD: examplepass
      WORDPRESS_DB_NAME: exampledb
    volumes:
      - wordpress:/var/www/html
  db:
    image: mysql:5.7
    restart: always
    environment:
      MYSQL_DATABASE: exampledb
      MYSQL_USER: exampleuser
      MYSQL_PASSWORD: examplepass
      MYSQL_RANDOM_ROOT_PASSWORD: '1'
    volumes:
      - db:/var/lib/mysql

volumes:
  wordpress:
  db:
```

Save and exit from the file. Now lets start the docker containers - `docker-compose up`. This will fire up the Wordpress and MySQL containers with appropriate configuration. Open a browser and type the URL - [`http://localhost:8080`](http://localhost:8080) and press enter. At this point, we can see that we have now successfully opened up the Wordpress setup page.

In order to access the Wordpress app over domain name, we need to make an entry in the `/etc/hosts` files and add the following at the end of the file - `127.0.0.1  my-wordpress-blog.local`. After this, now we should be able to access - [`http://my-wordpress-blog.local:8080`](http://my-wordpress-blog.local:8080) in browser.

In order to have HTTPS in the local development environment, we will use a utility called [mkcert](https://github.com/FiloSottile/mkcert). In order to have mkcert, we first need to install the dependency - `libnss3-tools`. Open a terminal and run - `sudo apt install libnss3-tools -y`. Now lets download the pre-built mkcert binary from the [github releases page](https://github.com/FiloSottile/mkcert/releases). Download the appropriate binary. Since I am using Ubuntu on my develoment machine, so I will use [mkcert-v1.4.3-linux-amd64](https://github.com/FiloSottile/mkcert/releases/download/v1.4.3/mkcert-v1.4.3-linux-amd64). Download the binary file and move it to `/usr/local/bin`. We also need to make the file executable - `chmod +x mkcert-v1.4.3-linux-amd64`. Now lets create a softlink with name - `mkcert` - `ln -s mkcert-v1.4.3-linux-amd64 mkcert`. The first step is to become a valid Certificate Authority for local machine - `mkcert -install`. This will install the root CA for local machine.
