Haproxy Docker Container
===================

This container (based on Debian 8) runs a hapoxy 1.5 service.

ryslog is installed in order to have haproxy logs available in /var/log/haproxy.

Exposed ports are 80, 443, 8000.

You can copy the haproxy_example_kaliop.cfg file in your project and mount it as a volume in /etc/haproxy/haproxy.cfg : this configuration file will use linked 'web', 'varnish' and 'solr' containers as backends that will be accessible via corresponding 'backend' header on port 80.


How to build & run the container
--------------------------------

* Check out this repository in a directory somewhere, and execute commands within it 
* If you are working behind a corporate http proxy, run the [klabs/forgetproxy container](https://registry.hub.docker.com/u/klabs/forgetproxy/)
* Build the image

``` sh
docker build -t haproxy .
```

If the build fails when fetching APT repositories/packages , try to build the image without cache :
    
``` sh
docker build --no-cache -t haproxy .
```

## How to use docker-compose to run container

``` sh
docker-compose up -d
``` 


Example custom docker-compose.yml file using haproxy container
----------------------------------------------------------


``` yml
varnish:
  extends:
    file: /home/user/docker/varnish4/docker-compose.yml
    service: varnish4
  image: kaliop_varnish4
  links:
  - web
  
web:
  privileged: true
  image: kaliop_webserver
  volumes:
   - .:/home/site/www/
   - "$SSH_AUTH_SOCK:/ssh-agent"
   - /var/run/docker.sock:/var/run/docker.sock
   - /usr/bin/docker:/usr/bin/docker
  environment:
   - DEV_UID=65527
   - DEV_GID=513
   - SSH_AUTH_SOCK=/ssh-agent
  cap_add:
    - SYS_PTRACE

haproxy:
  extends:
    file: /home/user/docker/haproxy/docker-compose.yml
    service: haproxy
  image: kaliop_haproxy
  volumes:
   - /home/user/docker/haproxy/haproxy_example_kaliop.cfg:/etc/haproxy/haproxy.cfg
  links:
    - web
    - varnish
    - solr

solr:
  extends:
    file: /home/user/docker/solr/docker-compose.yml
    service: solr
  image: kaliop_solr


``` 