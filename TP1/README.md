# DOCKER

## I. Init

### 3. sudo c pa bo

Ajouter votre utilisateur au groupe docker
```
sudo usermod -aG docker $USER

cat /etc/group | grep docker
docker:x:991:alexandre

docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

### 4. Un premier conteneur en vif

Lancer un conteneur NGINX
```
docker run -d -p 9999:80 nginx
```

🌞 Visitons

```
. Vérifier que le conteneur est actif avec une commande qui liste les conteneurs en cours de fonctionnement :
    docker ps
    46b44484d82a   nginx     "/docker-entrypoint.…"   2 minutes ago   Up 2 minutes   0.0.0.0:9999->80/tcp, [::]:9999->80/tcp   recursing_brown

. Afficher les logs du conteneur :
    docker logs recursing_brown
        /docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
        /docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
        /docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
        10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
        10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
        /docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
        /docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
        /docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
        /docker-entrypoint.sh: Configuration complete; ready for start up
        2024/12/11 09:56:06 [notice] 1#1: using the "epoll" event method
        2024/12/11 09:56:06 [notice] 1#1: nginx/1.27.3
        2024/12/11 09:56:06 [notice] 1#1: built by gcc 12.2.0 (Debian 12.2.0-14)
        2024/12/11 09:56:06 [notice] 1#1: OS: Linux 5.14.0-284.11.1.el9_2.x86_64
        2024/12/11 09:56:06 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1073741816:1073741816
        2024/12/11 09:56:06 [notice] 1#1: start worker processes
        2024/12/11 09:56:06 [notice] 1#1: start worker process 29

. Afficher toutes les informations relatives au conteneur avec une commande docker inspect:

    docker inspect recursing_brown
        [
            {
                "Id": "46b44484d82af61c56ff9d84124ef5024077942fba9b31098a73b6fd63a02fa3",
                "Created": "2024-12-11T09:56:00.366092606Z",
                "Path": "/docker-entrypoint.sh",
                "Args": [
                    "nginx",
                    "-g",
                    "daemon off;"
                ],
                "State": {
                    "Status": "running",
                    "Running": true,
                    "Paused": false,
                    "Restarting": false,
                    "OOMKilled": false,
                    "Dead": false,
                    "Pid": 3882,
                    "ExitCode": 0,
                    "Error": "",
                    "StartedAt": "2024-12-11T09:56:04.034577896Z",
                    "FinishedAt": "0001-01-01T00:00:00Z"
                },
                "Image": "sha256:66f8bdd3810c96dc5c28aec39583af731b34a2cd99471530f53c8794ed5b423e",
            }
        ] *je n'ais pas tout mis la liste est un peu trop longue !

. Afficher le port en écoute sur la VM avec un sudo ss -lnpt
    sudo ss -lnpt
        State   Recv-Q  Send-Q     Local Address:Port     Peer Address:Port  Process
        LISTEN  0       128              0.0.0.0:22            0.0.0.0:*      users:(("sshd",pid=682,fd=3))
        LISTEN  0       4096             0.0.0.0:9999          0.0.0.0:*      users:(("docker-proxy",pid=3828,fd=4))
        LISTEN  0       128                 [::]:22               [::]:*      users:(("sshd",pid=682,fd=4))
        LISTEN  0       4096                [::]:9999             [::]:*      users:(("docker-proxy",pid=3834,fd=4))

. Ouvrir le port 9999/tcp (vu dans le ss au dessus normalement) dans le firewall de la VM
        sudo firewall-cmd --state
    running
    
        sudo firewall-cmd --zone=public --add-port=9999/tcp --permanent
    success

        sudo firewall-cmd --reload
    success

        sudo firewall-cmd --zone=public --list-ports
    9999/tcp
```

🌞 On va ajouter un site Web au conteneur NGINX

. Lancez le conteneur avec la commande en dessous :
```
    docker rm -f recursing_brown
    recursing_brown

    docker run -d -p 9999:8080 -v /home/alexandre/nginx/index.html:/var/www/html/index.html -v /home/alexandre/nginx/site_nul.conf:/etc/nginx/conf.d/site_nul.conf nginx
    2e0699326acb432d192d9bb346c1031836d12e572353e9e025fcb112fe025163
```

🌞 Visitons
```
    docker ps
    CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS
                            NAMES
    2e0699326acb   nginx     "/docker-entrypoint.…"   3 minutes ago   Up 3 minutes   80/tcp, 0.0.0.0:9999->8080/tcp, [::]:9999->8080/tcp   bold_blackburn


    aucun port firewall à ouvrir : on écoute toujours port 9999 sur la machine hôte (la VM)
        sudo firewall-cmd --zone=public --list-ports
        9999/tcp
```

### 5. Un deuxième conteneur en vif

🌞 Lance un conteneur Python, avec un shell

```
    docker run -it python bash
    Unable to find image 'python:latest' locally
    latest: Pulling from library/python
    fdf894e782a2: Pull complete
    5bd71677db44: Pull complete
    551df7f94f9c: Pull complete
    ce82e98d553d: Pull complete
    5f0e19c475d6: Pull complete
    abab87fa45d0: Pull complete
    2ac2596c631f: Pull complete
    Digest: sha256:220d07595f288567bbf07883576f6591dad77d824dce74f0c73850e129fa1f46
    Status: Downloaded newer image for python:latest

    pip install aiohttp
    Successfully installed aiohappyeyeballs-2.4.4 aiohttp-3.11.10 aiosignal-1.3.1 attrs-24.2.0 frozenlist-1.5.0 idna-3.10 multidict-6.1.0 propcache-0.2.1 yarl-1.18.3

    pip install aioconsole
    Successfully installed aioconsole-0.8.1
```

