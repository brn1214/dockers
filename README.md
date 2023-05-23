# Goal
1. For simplicity, you can use Python as a web server: “python -m http.server 8000”.
    - Add this command to the CMD instruction in the Dockerfile.
2. Create a Dockerfile based on "python:3.10-alpine", in which.
    - Create a directory “/app” and assign it as WORKDIR.
    - Add a file containing the text "Hello world" to it.
    - Ensure that the web server is launched on behalf of the user with “uid 1001”.
3. Build Docker image with tag “1.0.0”.
4. Run the Docker container and check that the web application is running.
5. Upload the image to Docker Hub.
6. Create a Kubernetes Deployment manifest that launches container from the created image.
    - Deployment name must be “web”.
    - The number of running replicas must be two.
    - Add the use of Probes.
7. Install manifest on the Kubernetes cluster.
8. Provide access to the web application inside the cluster and check its operation
    - Use the kubectl port-forward command: “kubectl port-forward --address 0.0.0.0 deployment/web 8080:8000”.
    - Go to http://127.0.0.1:8080/hello.html.
---
## Step 1
### Server.py
```py
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello world!'

```
## Step 2
Dockerfile in server

```Dockerfile
# Базовый image
FROM python:3.10-alpine

# Переменные, используемые для создания окружения, в котором запустится приложение
ARG USER=app 
ARG UID=1001
ARG GID=1001

# Установка фреймворка
RUN pip install --no-cache-dir Flask==2.2.* 

# Создание пользователя операционной системы и его домашнего каталога
RUN addgroup -g ${GID} -S ${USER} \
   && adduser -u ${UID} -S ${USER} -G ${USER} \
   && mkdir -p /app \
   && chown -R ${USER}:${USER} /app
USER ${USER}

# Переход в каталог /app
WORKDIR /app

# Переменные окружения, необходимые для запуска web-приложения
ENV FLASK_APP=server.py \
   FLASK_RUN_HOST="0.0.0.0" \ 
   FLASK_RUN_PORT="8000" \
   PYTHONUNBUFFERED=1

# Копирование кода приложения в домашний каталог
COPY --chown=$USER:$USER server.py /app

# Публикация порта, который прослушивается приложением
EXPOSE 8000

# Команда запуска приложения
CMD ["flask", "run"]
```
## Step 3
Creating a Docker image
```shell
docker build -t brn12/server:1.0.0 -t brn12/server:latest server
[+] Building 1.1s (10/10) FINISHED
 => [internal] load .dockerignore                                                    0.0s 
 => => transferring context: 2B                                                      0.0s 
 => [internal] load build definition from Dockerfile                                 0.0s 
 => => transferring dockerfile: 1.28kB                                               0.0s 
 => [internal] load build context                                                    0.0s 
 => => transferring context: 30B                                                     0.0s 
 => [1/5] FROM docker.io/library/python:3.10-alpine@sha256:def82962a6ee048e54b5bec2  0.0s 
 => CACHED [2/5] RUN pip install --no-cache-dir Flask==2.2.*                         0.0s 
 => CACHED [3/5] RUN addgroup -g 1001 -S app    && adduser -u 1001 -S app -G app     0.0s 
 => CACHED [4/5] WORKDIR /app                                                        0.0s 
 => CACHED [5/5] COPY --chown=app:app server.py /app                                 0.0s 
 => exporting to image                                                               0.0s 
 => => exporting layers                                                              0.0s 
 => => writing image sha256:a107a8446cb1c03d59488da7ab893208b4c09d3c86df788b35b280f  0.0s 
 => => naming to docker.io/brn12/server:1.0.0                                        0.0s 
 => => naming to docker.io/brn12/server:latest 
```
## Step 4
We run the image
```shell
docker run -ti --rm -p 8000:8000 --name server brn12/server:1.0.0
 * Serving Flask app 'server.py'
 * Debug mode: off
WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
 * Running on all addresses (0.0.0.0)
 * Running on http://127.0.0.1:8000
 * Running on http://172.17.0.2:8000
Press CTRL+C to quit
172.17.0.1 - - [23/May/2023 19:23:05] "GET / HTTP/1.1" 200 
```
And check in another terminal
```shell
PS C:\Users\pret_\OneDrive\Documentos\ITMO\Practice 4> curl http://127.0.0.1:8000


StatusCode        : 200
StatusDescription : OK
Content           : Hello world!
RawContent        : HTTP/1.1 200 OK
                    Connection: close
                    Content-Length: 12
                    Content-Type: text/html; charset=utf-8
                    Date: Tue, 23 May 2023 19:23:05 GMT
                    Server: Werkzeug/2.3.4 Python/3.10.11

                    Hello world!
Forms             : {}
Headers           : {[Connection, close], [Content-Length, 12], [Content-Type,
                    text/html; charset=utf-8], [Date, Tue, 23 May 2023 19:23:05 GMT]...}
Images            : {}
InputFields       : {}
Links             : {}
ParsedHtml        : mshtml.HTMLDocumentClass
RawContentLength  : 12
```
## Step 5
We access to the Docker account
```shell
PS C:\Users\pret_\OneDrive\Documentos\ITMO\Practice 4> docker login -u "brn12" -p "xxxxxxxxxxxxx" docker.io    
WARNING! Using --password via the CLI is insecure. Use --password-stdin.
Login Succeeded
```
Now we push the image
```shell
PS C:\Users\pret_\OneDrive\Documentos\ITMO\Practice 4> docker push brn12/server:1.0.0
The push refers to repository [docker.io/brn12/server]
129afa0c9bca: Pushed
5f70bf18a086: Layer already exists
6c3a76a450cd: Pushed
a881df45fae4: Pushed
f288e3d33a08: Layer already exists
00b0c44330bf: Layer already exists
3b787bd09f7a: Layer already exists
208977ac81d7: Layer already exists
bb01bd7e32b5: Layer already exists
1.0.0: digest: sha256:c6c1913659732934d23b0bd84ec5a70fd01f94b8f58c2feaf06536eef4dcdb76 size: 2200
```
![1](https://github.com/brn1214/dockers/assets/127547589/c1adf620-b3d5-4084-8ac3-5bd37c424b3c)

## Step 6
We copy the image
```shell
PS C:\Users\pret_\OneDrive\Documentos\ITMO\Practice 4> docker compose build
[+] Building 1.8s (11/11) FINISHED
 => [internal] load .dockerignore                                                    0.0s 
 => => transferring context: 2B                                                      0.0s 
 => [internal] load build definition from Dockerfile                                 0.0s 
 => => transferring dockerfile: 1.28kB                                               0.0s 
 => [internal] load metadata for docker.io/library/python:3.10-alpine                1.7s 
 => [auth] library/python:pull token for registry-1.docker.io                        0.0s 
 => [internal] load build context                                                    0.0s 
 => => transferring context: 30B                                                     0.0s 
 => [1/5] FROM docker.io/library/python:3.10-alpine@sha256:def82962a6ee048e54b5bec2  0.0s 
 => CACHED [2/5] RUN pip install --no-cache-dir Flask==2.2.*                         0.0s 
 => CACHED [3/5] RUN addgroup -g 1001 -S app    && adduser -u 1001 -S app -G app     0.0s 
 => CACHED [4/5] WORKDIR /app                                                        0.0s 
 => CACHED [5/5] COPY --chown=app:app server.py /app                                 0.0s 
 => exporting to image                                                               0.0s 
 => => exporting layers                                                              0.0s 
 => => writing image sha256:061467f54c641ff897c0e5505206cbdd32ebf7cf7742c1161d995cc  0.0s 
 => => naming to docker.io/brn12/server:1.0.0                                        0.0s 
[+] Building 0.5s (9/9) FINISHED
 => [internal] load .dockerignore                                                    0.0s 
 => => transferring context: 2B                                                      0.0s 
 => [internal] load build definition from Dockerfile                                 0.0s 
 => => transferring dockerfile: 926B                                                 0.0s 
 => [internal] load metadata for docker.io/library/python:3.10-alpine                0.5s 
 => [internal] load build context                                                    0.0s 
 => => transferring context: 31B                                                     0.0s 
 => [1/4] FROM docker.io/library/python:3.10-alpine@sha256:def82962a6ee048e54b5bec2  0.0s 
 => CACHED [2/4] RUN addgroup -g 1001 -S app     && adduser -u 1001 -S app -G app    0.0s 
 => CACHED [3/4] WORKDIR /app                                                        0.0s 
 => CACHED [4/4] COPY --chown=app:app client.py /app                                 0.0s 
 => exporting to image                                                               0.0s 
 => => exporting layers                                                              0.0s 
 => => naming to docker.io/brn12/client:1.0.0 
 ```
 And we run it
 ```shell
 PS C:\Users\pret_\OneDrive\Documentos\ITMO\Practice 4> docker compose up
[+] Running 3/3
 ✔ Container practice4-server-1                                      Created         0.1s 
 ! server Published ports are discarded when using host network mode                 0.0s 
 ✔ Container practice4-client-1                                      Created         0.1s 
Attaching to practice4-client-1, practice4-server-1
practice4-server-1  |  * Serving Flask app 'server.py'
practice4-server-1  |  * Debug mode: off
practice4-server-1  | WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
practice4-server-1  |  * Running on all addresses (0.0.0.0)
practice4-server-1  |  * Running on http://127.0.0.1:8000
practice4-server-1  |  * Running on http://192.168.65.4:8000
practice4-server-1  | Press CTRL+C to quit
```
Finally we check the logs
 ```shell
PS C:\Users\pret_\OneDrive\Documentos\ITMO\Practice 4> docker logs practice4-server-1     
 * Serving Flask app 'server.py'
 * Debug mode: off
WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
 * Running on all addresses (0.0.0.0)
 * Running on http://127.0.0.1:8000
 * Running on http://192.168.65.4:8000
Press CTRL+C to quit
127.0.0.1 - - [23/May/2023 19:30:01] "GET / HTTP/1.1" 200 -
127.0.0.1 - - [23/May/2023 19:30:17] "GET / HTTP/1.1" 200 -
 * Serving Flask app 'server.py'
 * Debug mode: off
WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
 * Running on all addresses (0.0.0.0)
 * Running on http://127.0.0.1:8000
 * Running on http://192.168.65.4:8000
Press CTRL+C to quit
127.0.0.1 - - [23/May/2023 19:33:28] "GET / HTTP/1.1" 200 -
127.0.0.1 - - [23/May/2023 19:33:34] "GET / HTTP/1.1" 200 -
127.0.0.1 - - [23/May/2023 19:33:35] "GET / HTTP/1.1" 200 -
127.0.0.1 - - [23/May/2023 19:33:37] "GET / HTTP/1.1" 200 -
127.0.0.1 - - [23/May/2023 19:33:38] "GET / HTTP/1.1" 200 -
127.0.0.1 - - [23/May/2023 19:33:39] "GET / HTTP/1.1" 200 -
 ```shell
