# 🌞 Installer Docker votre machine Azure

- uninstall all conflicting packages 

* for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done

- Set up Docker's apt repository.

* sudo apt-get update

* sudo apt-get install ca-certificates curl

* sudo install -m 0755 -d /etc/apt/keyrings

* sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc

* sudo chmod a+r /etc/apt/keyrings/docker.asc

 - Add the repository to Apt sources:

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

* sudo apt-get update

- To install the latest version, run:

sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

- Verify that the installation is successful by running the hello-world image:

* sudo docker run hello-world

* sudo docker info 

* sudo systemctl start docker

* sudo usermod -aG docker $(whoami)

* sudo systemctl restart docker

# 🌞 Utiliser la commande docker run

* docker run --name web -d -p 9999:80 nginx

 * curl http://localhost:9999

<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>

# 🌞 Rendre le service dispo sur internet

*  curl http://135.236.10.211:9999
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>

# 🌞 Custom un peu le lancement du conteneur

* mkdir -p ~/nginx_custom/{conf,html}

* cat <<EOF > ~/nginx_custom/conf/custom.conf
server {
  listen 7777;
  root /var/www/tp_docker;
}
EOF

* echo "<h1>Cest un site web ce truc </h1>" > ~/nginx_custom/html/index.html

* docker run --name meow -d \
  -p 9999:7777 \
  -v ~/nginx_custom/conf/custom.conf:/etc/nginx/conf.d/custom.conf \
  -v ~/nginx_custom/html:/var/www/tp_docker \
  --memory=512m \
  nginx


## PART2

# 🌞 Construire votre propre image

* touch Dockerfile 

* nano index.html 
body {
    font-family: Arial, sans-serif;
    text-align: center;
    background-color: #f4f4f4;
    margin: 0;
    padding: 0;
}

.container {
    padding: 50px;
}

h1 {
    color: #ff6347;
    font-size: 36px;
}

p {
    font-size: 18px;
    color: #333;
}

button {
    padding: 15px 30px;
    font-size: 18px;
    background-color: #4CAF50;
    color: white;
    border: none;
    border-radius: 5px;
    cursor: pointer;
    margin-top: 20px;
}

button:hover {
    background-color: #45a049;
}

.hidden {
    display: none;
}

#trollContent h2 {
    color: #ff6347;
    font-size: 30px;
}

#trollContent p {
    color: #555;
}

#trollContent img {
    margin-top: 20px;
    width: 200px;
    height: 200px;
}

azureuser@TP1:~$ ls
Dockerfile  index.html  myproject
azureuser@TP1:~$ cat Dockerfile
FROM ubuntu

RUN apt update -y

RUN apt install -y apache2

RUN mkdir /etc/apache2/logs

COPY apache2.conf /etc/apache2/

RUN echo "ServerName localhost" >> /etc/apache2/apache2.conf

COPY index.html /var/www/html/

CMD [ "apache2", "-DFOREGROUND" ]

* nano apache2.conf

Listen 80

LoadModule mpm_event_module "/usr/lib/apache2/modules/mod_mpm_event.so"
LoadModule dir_module "/usr/lib/apache2/modules/mod_dir.so"
LoadModule authz_core_module "/usr/lib/apache2/modules/mod_authz_core.so"


DirectoryIndex index.html

DocumentRoot "/var/www/html/"

ErrorLog "logs/error.log"
LogLevel warn

* cat Dockerfile

FROM ubuntu

RUN apt update -y

RUN apt install -y apache2

RUN mkdir /etc/apache2/logs

COPY apache2.conf /etc/apache2/

RUN echo "ServerName localhost" >> /etc/apache2/apache2.conf

COPY index.html /var/www/html/

CMD [ "apache2", "-DFOREGROUND" ]

* docker build . -t project

* docker run -p 9999:80 project

* sudo docker run -d -p 9999:80 project

# 🌞 Installez un WikiJS en utilisant Docker

* touch docker-compose.yml 
nano docker-compose.yml 

version: "3.8"

services:
  db:
    image: postgres:13
    restart: always
    environment:
      POSTGRES_DB: wikijs
      POSTGRES_USER: wikijs
      POSTGRES_PASSWORD: mysecretpassword
    volumes:
      - db_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U wikijs"]
      interval: 10s
      retries: 5
      timeout: 5s

  wiki:
    image: requarks/wiki:latest
    restart: always
    depends_on:
      db:
        condition: service_healthy
    ports:
      - "80:3000"
    environment:
      DB_TYPE: postgres
      DB_HOST: db
      DB_PORT: 5432
      DB_USER: wikijs
      DB_PASS: mysecretpassword
      DB_NAME: wikijs

volumes:
  db_data:

## 🌞 Vous devez :

construire une image qui

contient python3

contient l'application et ses dépendances
lance l'application au démarrage du conteneur


écrire un docker-compose.yml qui définit le lancement de deux conteneurs :

l'app python
le Redis dont il a besoin



* mkdir my-meow-app
* cd my-meow-app
* nano Dockerfile

FROM python:3.9-slim

WORKDIR /app

COPY . /app

RUN pip install --no-cache-dir -r requirements.txt
RUN pip install gunicorn

EXPOSE 8888

CMD ["python", "app.py"]


* nano requirements.txt

flask==2.1.1
Werkzeug==2.0.3
redis==4.0.2

* nano app.py 

import redis
import time
import socket

time.sleep(5)

r = redis.StrictRedis(host='redis-db', port=6379, db=0)

from flask import Flask, request, render_template
app = Flask(__name__)


@app.route('/')
@app.route('/index')
def index():
    hostname=socket.gethostname()
    return render_template('index.html',
                           title='Home',
                           container_hostname=hostname)

@app.route('/add', methods=['POST', 'GET'])
def add():
    if request.method == 'POST':
        r.set(request.form['key'], request.form['value'])

    return 'Successfully added key ' + request.form['key']

@app.route('/get', methods=['POST'])
def get():
    try:
        if request.method == 'POST':
            keyBytes = r.get(request.form['key'])
            key = keyBytes.decode('utf-8')
        return 'You asked about key ' + request.form['key'] + ". Value : " + key
    except:
        return 'Key ' + request.form['key'] + " does not exist."


if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8888)

* mkdir templates
* cd templates
* nano index.html

<h1>Add key</h1>
<form action="{{ url_for('add') }}" method = "POST">
Key:
<input type="text" name="key" >
Value:
<input type="text" name="value" >
<input type="submit" value="Submit">
</form>

<h1>Check key</h1>
<form action="{{ url_for('get') }}" method = "POST">

Key:
<input type="text" name="key" >
<input type="submit" value="Submit">
</form>

Host : {{ container_hostname }}

* nano docker-compose.yml

version: "3.8"

services:
  app:
    build: .
    ports:
      - "8888:8888"
    depends_on:
      redis:
        condition: service_healthy  # Assurer que Redis soit healthy avant le lancement
    environment:
      - REDIS_HOST=redis
      - REDIS_PORT=6379
    networks:
      - app-network

  redis:
    image: redis:latest
    container_name: redis-db
    ports:
      - "6379:6379"
    networks:
      - app-network
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]  # Vérifier que Redis répond au ping
      interval: 10s
      retries: 5
      start_period: 10s

networks:
  app-network:
    driver: bridge

* docker compose build
* sudo docker compose up

WARN[0000] /home/azureuser/wikijs-docker/my-meow-app/docker-compose.yml: the attribute `version` is obsolete, it will be ignored, please remove it to avoid potential confusion
[+] Running 2/2
 ✔ Container redis-db           Created                                                                                                                                                    0.0s
 ✔ Container my-meow-app-app-1  Created                                                                                                                                                    0.0s
Attaching to app-1, redis-db
redis-db  | 1:C 31 Mar 2025 22:06:42.113 # WARNING Memory overcommit must be enabled! Without it, a background save or replication may fail under low memory condition. Being disabled, it can also cause failures without low memory condition, see https://github.com/jemalloc/jemalloc/issues/1328. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
redis-db  | 1:C 31 Mar 2025 22:06:42.114 * oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
redis-db  | 1:C 31 Mar 2025 22:06:42.114 * Redis version=7.4.2, bits=64, commit=00000000, modified=0, pid=1, just started
redis-db  | 1:C 31 Mar 2025 22:06:42.114 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
redis-db  | 1:M 31 Mar 2025 22:06:42.115 * monotonic clock: POSIX clock_gettime
redis-db  | 1:M 31 Mar 2025 22:06:42.116 * Running mode=standalone, port=6379.
redis-db  | 1:M 31 Mar 2025 22:06:42.118 * Server initialized
redis-db  | 1:M 31 Mar 2025 22:06:42.119 * Loading RDB produced by version 7.4.2
redis-db  | 1:M 31 Mar 2025 22:06:42.119 * RDB age 1174 seconds
redis-db  | 1:M 31 Mar 2025 22:06:42.119 * RDB memory usage when created 0.93 Mb
redis-db  | 1:M 31 Mar 2025 22:06:42.119 * Done loading RDB, keys loaded: 0, keys expired: 0.
redis-db  | 1:M 31 Mar 2025 22:06:42.120 * DB loaded from disk: 0.001 seconds
redis-db  | 1:M 31 Mar 2025 22:06:42.120 * Ready to accept connections tcp
app-1     |  * Serving Flask app 'app' (lazy loading)
app-1     |  * Environment: production
app-1     |    WARNING: This is a development server. Do not use it in a production deployment.
app-1     |    Use a production WSGI server instead.
app-1     |  * Debug mode: off
app-1     |  * Running on all addresses.
app-1     |    WARNING: This is a development server. Do not use it in a production deployment.
app-1     |  * Running on http://172.19.0.3:8888/ (Press CTRL+C to quit)
app-1     | 31.34.146.37 - - [31/Mar/2025 22:06:58] "GET / HTTP/1.1" 200 -

# 🌞 Prouvez que vous pouvez devenir root

