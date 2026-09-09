# Phase 9 — Containers & Automation

## 1. Objectives

The goal of this phase was to introduce Linux containers with Podman and build a small, realistic multi-container application.

The phase covered container fundamentals and lifecycle, rootless/daemonless operation, images, port publishing, bind mounts, named volumes, logs, environment variables, Containerfiles, custom networks, PostgreSQL persistence, a Flask CRUD API, and basic shell automation.

The focus remained on practical Linux/container administration rather than application development.

## 2. Podman Fundamentals

Podman was used on the Linux Mint host.

The first test:

```bash
podman run --rm alpine uname -a
```

demonstrated that the Alpine container had its own userspace while sharing the Linux Mint kernel.

Core model:

```text
VM        = own kernel + userspace
Container = shared host kernel + isolated userspace/processes
```

Important lifecycle commands:

```bash
podman images
podman ps
podman ps -a
podman run
podman exec
podman stop
podman start
podman rm
```

A persistent test container was created with:

```bash
podman run -d --name alpine-lab alpine sleep 3600
podman exec -it alpine-lab /bin/sh
```

The container remained alive while its PID 1 process remained running. `--rm` was used for disposable one-shot containers.

Rootless operation means containers can run under a normal user account. Daemonless operation means Podman does not require a permanent central daemon comparable to the traditional Docker `dockerd` architecture.

## 3. Port Publishing

An nginx container was published with:

```bash
podman run -d --name web-lab   -p 8080:80   docker.io/library/nginx:latest
```

The mapping:

```text
0.0.0.0:8080 -> 80/tcp
```

means:

```text
Linux Mint :8080 -> Podman -> container :80
```

`0.0.0.0` means the service is published on all IPv4 interfaces of the Mint host.

The service was verified with `curl`.

## 4. Bind Mounts

A host directory was mounted into nginx:

```bash
mkdir -p ~/container-web

podman run -d --name web-lab   -p 8080:80   -v ~/container-web:/usr/share/nginx/html:ro   docker.io/library/nginx:latest
```

The `:ro` option made the content read-only from inside the container.

Changes to files in `~/container-web` appeared immediately through nginx. The host files survived container deletion.

A bind mount therefore gives the container access to an explicitly selected host path.

## 5. Named Volumes

A Podman-managed volume was created:

```bash
podman volume create web-data
podman volume inspect web-data
```

It was mounted with:

```bash
-v web-data:/usr/share/nginx/html
```

The nginx container was deleted and recreated with the same volume, proving that the stored content survived independently.

Core model:

```text
container = disposable runtime
volume    = persistent data
```

Unlike a bind mount, Podman chooses and manages the host storage location for a named volume.

## 6. Logs

Logs were inspected with:

```bash
podman logs nginx-web
podman logs python-app
```

The Flask logs later showed the CRUD requests and HTTP status codes:

```text
GET     /notes   -> 200
POST    /notes   -> 201
PUT     /notes/2 -> 200
DELETE  /notes/2 -> 204
```

This demonstrated access to application stdout/stderr without entering the container.

## 7. Environment Variables

Runtime configuration was introduced with `-e`:

```bash
podman run --rm   -e LAB_ENV=production   -e APP_PORT=8080   alpine env
```

Core model:

```text
image       = reusable software
environment = runtime configuration
```

Environment variables were later used to configure PostgreSQL and the Flask application.

## 8. Custom Images and Containerfiles

A custom nginx image was built from a `Containerfile`:

```dockerfile
FROM docker.io/library/nginx:latest
COPY index.html /usr/share/nginx/html/index.html
```

Build:

```bash
podman build -t my-nginx-web .
```

This demonstrated content baked into an image rather than supplied through external storage.

The Python application later used:

```dockerfile
FROM docker.io/library/python:3.13-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

CMD ["python", "app.py"]
```

Dependencies:

```text
flask
psycopg[binary]
```

Build:

```bash
podman build -t my-python-app .
```

## 9. Container Networking

A dedicated network was created:

```bash
podman network create app-net
podman network ls
```

Containers on `app-net` successfully communicated using container names instead of hard-coded IP addresses.

For example:

```bash
ping -c 2 my-web
wget -qO- http://my-web
```

The network used the `10.89.0.0/24` range during the lab.

Core model:

```text
container A -> app-net -> container B
```

Service names such as `postgres-db` are preferable to hard-coded container IP addresses, which may change.

## 10. PostgreSQL Container

A persistent database volume was created:

```bash
podman volume create postgres-data
```

PostgreSQL was launched on `app-net`:

```bash
podman run -d   --name postgres-db   --network app-net   -e POSTGRES_DB=labdb   -e POSTGRES_USER=labuser   -e POSTGRES_PASSWORD=labpass   -v postgres-data:/var/lib/postgresql   docker.io/library/postgres:latest
```

The PostgreSQL 18 image used in the lab required the persistent mount at `/var/lib/postgresql`, with its version-specific data layout beneath that directory.

PostgreSQL port 5432 was deliberately not published to Linux Mint. Database access remained internal to the Podman network.

Local database access:

```bash
podman exec -it postgres-db psql -U labuser -d labdb
```

TCP access from another container:

```bash
podman run --rm -it   --network app-net   -e PGPASSWORD=labpass   docker.io/library/postgres:latest   psql -h postgres-db -U labuser -d labdb
```

This proved service-to-service TCP connectivity over `app-net`.

## 11. Database Persistence

A simple table was created:

```sql
CREATE TABLE notes (
    id SERIAL PRIMARY KEY,
    text VARCHAR(100)
);
```

A row was inserted:

```sql
INSERT INTO notes (text)
VALUES ('Persistent data fron PostgreSQL volume');
```

The `postgres-db` container was stopped and removed, then recreated with the same `postgres-data` volume.

After recreation:

```sql
SELECT * FROM notes;
```

still returned the stored row.

This proved that the database state belonged to the volume rather than the disposable container.

## 12. Flask + PostgreSQL Application

A small Flask application used psycopg to connect to PostgreSQL.

Database configuration was read from environment variables:

```python
def get_connection():
    return psycopg.connect(
        host=os.environ["DB_HOST"],
        dbname=os.environ["DB_NAME"],
        user=os.environ["DB_USER"],
        password=os.environ["DB_PASSWORD"],
    )
```

Flask listened on:

```python
app.run(host="0.0.0.0", port=5000)
```

The application container was launched with:

```bash
podman run -d   --name python-app   --network app-net   -p 5000:5000   -e DB_HOST=postgres-db   -e DB_NAME=labdb   -e DB_USER=labuser   -e DB_PASSWORD=labpass   localhost/my-python-app:latest
```

Application path:

```text
Client
  |
Linux Mint :5000
  |
Podman port publishing
  |
python-app / Flask :5000
  |
app-net
  |
postgres-db :5432
  |
postgres-data
```

## 13. CRUD API

The Flask application implemented a minimal CRUD API.

### READ

```text
GET /notes
```

```bash
curl http://127.0.0.1:5000/notes
```

### CREATE

```text
POST /notes
```

```bash
curl -X POST http://127.0.0.1:5000/notes   -H 'Content-Type: application/json'   -d '{"text":"Created through the API"}'
```

The API returned HTTP `201`.

### UPDATE

```text
PUT /notes/<id>
```

```bash
curl -X PUT http://127.0.0.1:5000/notes/2   -H 'Content-Type: application/json'   -d '{"text":"Updated through the API"}'
```

### DELETE

```text
DELETE /notes/<id>
```

```bash
curl -i -X DELETE http://127.0.0.1:5000/notes/2
```

The API returned:

```text
HTTP/1.1 204 NO CONTENT
```

A final GET confirmed that note 2 was deleted.

This completed a full:

```text
CREATE -> READ -> UPDATE -> DELETE
```

cycle through the containerized application.

## 14. Integration with the KVM Lab

The Flask application was published as:

```text
0.0.0.0:5000 -> 5000/tcp
```

Linux Mint already had:

```text
virbr10 = 10.10.10.254/24
```

The application was therefore reachable from the existing isolated KVM network at:

```text
10.10.10.254:5000
```

From Alpine-Lab-01:

```bash
curl http://10.10.10.254:5000/notes
```

returned HTTP `200 OK` and PostgreSQL-backed JSON data. Alpine-Lab-02 also successfully reached the API.

Because `10.10.10.254` is on the same `10.10.10.0/24` subnet as the Alpine guests, traffic to it is local-subnet traffic and does not need Alpine-Lab-01 to route it.

The container itself is not on `10.10.10.0/24`. It remains on Podman's separate `app-net`; Linux Mint publishes the container service through its own interfaces.

## 15. Security Observations

The setup was intentionally simple for the lab:

- Flask was exposed through `0.0.0.0:5000`.
- The CRUD API had no authentication.
- Database credentials were supplied through environment variables.
- Flask used the Werkzeug development server.
- PostgreSQL port 5432 was not published to the Mint host.

The Flask warning that its development server should not be used in production was expected. Production WSGI deployment and application hardening were intentionally outside this phase's scope.

## 16. Basic Automation

A simple startup script was created:

```sh
#!/bin/sh

echo "Starting PostgreSQL..."
podman start postgres-db

echo "Starting Python API..."
podman start python-app

echo
echo "Running containers:"
podman ps
```

It was made executable:

```bash
chmod +x start-stack.sh
```

The stack could then be started with:

```bash
./start-stack.sh
```

After stopping the containers, the script successfully restarted both services. A subsequent:

```bash
curl http://127.0.0.1:5000/notes
```

returned the persistent database row.

This introduced basic repeatable automation without adding a separate orchestration platform.

## 17. Final Architecture

```text
                         KVM LAB
                    10.10.10.0/24
                           |
          +----------------+----------------+
          |                |                |
    Alpine-Lab-01    Alpine-Lab-02    Alpine-Lab-03
      10.10.10.1       10.10.10.2       10.10.10.3
          |                |                |
          +----------------+----------------+
                           |
                         virbr10
                           |
                   Linux Mint host
                     10.10.10.254
                           |
                     TCP/5000 publish
                           |
                           v
                   +---------------+
                   |  python-app   |
                   | Flask :5000   |
                   +-------+-------+
                           |
                         app-net
                     10.89.0.0/24
                           |
                           v
                   +---------------+
                   |  postgres-db  |
                   |   :5432       |
                   +-------+-------+
                           |
                           v
                    postgres-data
                   persistent volume
```

## 18. Key Commands

| Purpose | Command |
|---|---|
| List images | `podman images` |
| Running containers | `podman ps` |
| All containers | `podman ps -a` |
| Run container | `podman run` |
| Execute in container | `podman exec -it CONTAINER COMMAND` |
| Stop container | `podman stop CONTAINER` |
| Start container | `podman start CONTAINER` |
| Remove container | `podman rm CONTAINER` |
| Inspect container | `podman inspect CONTAINER` |
| View logs | `podman logs CONTAINER` |
| Build image | `podman build -t IMAGE .` |
| Create volume | `podman volume create NAME` |
| List volumes | `podman volume ls` |
| Inspect volume | `podman volume inspect NAME` |
| Create network | `podman network create NAME` |
| List networks | `podman network ls` |
| Publish port | `-p HOST_PORT:CONTAINER_PORT` |
| Bind mount | `-v HOST_PATH:CONTAINER_PATH` |
| Named volume | `-v VOLUME:CONTAINER_PATH` |
| Environment variable | `-e NAME=value` |
| Start all containers | `podman start --all` |
| Stop all containers | `podman stop --all` |

## 19. Key Lessons Learned

- Containers share the host kernel while isolating processes and userspace.
- Images are reusable templates; containers are instances.
- Containers can be disposable while volumes preserve important state.
- Bind mounts expose chosen host paths; named volumes are managed by Podman.
- Port publishing maps host ports to container ports.
- `0.0.0.0` publishes on all host IPv4 interfaces.
- Custom networks provide container-to-container communication and name resolution.
- Internal services such as PostgreSQL do not need host port publishing.
- Environment variables separate runtime configuration from images.
- Containerfiles make application images reproducible.
- `podman logs` exposes application output from the host.
- PostgreSQL data survived complete database-container recreation.
- The Flask API, PostgreSQL database, Podman network, and persistent volume formed a working multi-container application.
- Simple shell scripting made startup repeatable without introducing orchestration.

## 20. Phase Result

Phase 9 produced a working container environment consisting of:

- Custom nginx and Python images
- A Flask `python-app` container
- A PostgreSQL `postgres-db` container
- A persistent `postgres-data` named volume
- A dedicated `app-net` container network
- TCP/5000 host publishing
- A functional CRUD REST API
- Connectivity from the existing KVM isolated network
- Container logging
- Basic startup automation

The phase progressed from container fundamentals to a small persistent multi-container application while keeping the scope focused on Linux and container administration.
