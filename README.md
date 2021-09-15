#Getting Started:
The easiest way to use this guide is to download the git repo here:
https://github.com/seth-macpherson/nginx-for-assets

Once you've downloaded the repo, you have the following file structure:

#Run it!
Build the docker containers and boot it up with:

docker-compose up --build

#See it! http://localhost:5000


#How does it work?
Before we get into the nitty-gritty of NGINX configuration settings let's take a look at how we're building the containers.

If you look below you'll see two files: docker-compose.yml and Dockerfile.nginx

Read these carefully:

## docker-compose.yml

```
  ###works
  version: '3.5'
  services:
    # This section should be replaced by YOUR web app
    # https://github.com/tobilg/docker-mini-webserver
    # provides a lightweight Express.JS web server on port 3000
    app:
      image: tobilg/mini-webserver
      container_name: app
      volumes:
        - ./example-app:/app/public:cached
      networks:
        - local-network

    # Test this NGINX and the sample app above
    # By opening http://localhost:5000
    # If working, you should see "NGINX + Reverse Proxy Guide"
    nginx:
      build:
        context: .
        dockerfile: Dockerfile.nginx
      environment:
        # This corresponds to the port in use by your app (above)
        - WEB_APP_PORT=3000
        # This is the hostname which also corresponds to the container_name (above)
        - WEB_APP_HOSTNAME=app
      container_name: nginx
      volumes:
        # Static assets are mapped here:
        - ./nginx/assets:/app/static/www:cached
        - ./nginx/static/errors:/app/static/errors:cached
        # Static (html, robots.txt, etc.) files can be placed here
        # - ./nginx/local/index.html:/app/static/errors/50x.html:cached
      networks:
        - local-network
      depends_on:
        - app
      ports:
        # We're going to access NGINX as http://localhost:5000
        - "5000:3000"
  networks:
    local-network:
      name: my-local-network

```

# NGINX Dockerfile (alpine build)
```
  # Start with the smallest NGINX build we can (1.21.3 is current as of Sept 2021)
  FROM nginx:1.21.3-alpine
  # Enable access log output via stdout
  ENV NGINX_ACCESS_LOG=yes
  # This is where all the configuration settings come together
  # By copying to /etc/nginx/templates/ we get ENV variable substitution
  # which makes our configuration a lot easier to manage from Docker
  COPY nginx/templates/default.conf.template /etc/nginx/templates/
  # This is THE NGINX config that sets global settings and, honestly,
  # it probably isn't necessary to modify this file but I like the option
  COPY nginx/templates/nginx.conf /etc/nginx/
  # If it's static, copy it over
  COPY nginx/static /app/static
  # This folder is used by NGINX for disk-based cache
  RUN mkdir -p /var/cache/nginx/static_cache
  RUN chown nginx:nginx /var/cache/nginx/static_cache
```
