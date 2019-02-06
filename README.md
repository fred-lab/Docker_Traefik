# Docker Traefik
If you run multiple project with Docker on your dev environment, you might be have conflict with containers running on the same port (eg multiple Nginx container on port 80) Traefik is a reverse proxy, we could use it on a dev environment to redirect the trafic to each project's container, and create hostname for each dev environment.

## Requirement
- Docker
- Docker-compose

## How to use
### Launch Traefik container
- Clone this repo
- Rename **docker-compose.yml.dist** to **docker-compose.yml**
- Run ```docker-compose up -d```
- Go to "localhost:8080" in a web browser, you will see Traefik's dashboard
- Enjoy

Note : After this step, Traefik will start when your PC start (restart: always) and it's no need to rerun ```docker-compose up -d```

### Update your projet
Each project must have a docker-compose.yml

Traefik use Label to interact with containers.

You should use this labels for each services you want to connect with Traefik

**Labels :**
- **traefik.port=80** : (required) Tells Traefik which port to return traffic to. If your Nginx / Apache container liston on port 80, specify 80

- **traefik.frontend.rule=Host:example_hostname.localhost,www.example_hostname.localhost** : (required) Replace *example_hostname* by your own hostname for your dev environment. Simply go to your web browser, with this hostname, and Traefik will "reverse-proxy" to the correct container and serve your application. The extension *.localhost avoids having to edit the /etc/host file, see [this article](https://ma.ttias.be/chrome-force-dev-domains-https-via-preloaded-hsts/) for more information

- **traefik.enable=true** : (required) Tell Traefik to monitor this container. By default, Traefik monitor every containers on the host. To keep it clean, Traefik is set to not watch every container unless they have this label set to true. For many case, we just want to monitor the Nginx / Apache container.

- **traefik.frontend.entryPoints=http,https** : (optionnal) This label is used to specify wich protocol you want to use

- **traefik.backend=** *backend_name* : (optionnal) If you want to do **load-balancing** with some of your containers, use this label.

- **traefik.docker.network=proxy** : (optionnal) If you want to use Traefik in a specified network (unless of network_mode: host) use this label. The name of the network must be the same you have in your Traefik's docker-compose.yml

####Example with an Nginx container, with the minimum configuration :

```services:
   
      front:
          image: nginx
          volumes:
              - .:/home/docker:ro
              - ./docker/front/default.conf:/etc/nginx/conf.d/default.conf:ro
          labels:
              - traefik.port=80
              - traefik.enable=true
              - traefik.frontend.rule=Host:traefik.localhost,www.traefik.localhost
  ```

Note : since Traefik handle port forwarding, it is not used to expose the port 80 of the Nginx container

If you go to localhost:8080, you will see the Traefik's dashbord with all managed container

 ![traefik dashbord](https://docs.traefik.io/img/web.frontend.png)




