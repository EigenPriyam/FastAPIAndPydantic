# 06 — Docker

**Docker = build, share and run containers.** Analogy: Maggi (noodles + masala packed together) -> same taste for every user. Docker packs code + everything it needs -> runs the same everywhere.

## Problems Docker solves
1. **Consistency** — different OS/library versions between dev and prod cause bugs.
2. **Isolation** — multiple websites on one server stay isolated in containers.
3. **Scalability** — easy to run the same code on many machines.

## How it works
- **Docker Engine** creates and runs containers.
- **Docker CLI (client)** --REST API--> **Docker Daemon (server)** does the actual work.
- Main objects: **Images, Containers, Volumes, Networks**.

## Key terms
| Term | Meaning |
|---|---|
| **Dockerfile** | Instructions to build an image |
| **Image** | Read-only blueprint: base image (OS/Python) + code + exact dependencies |
| **Container** | A *running instance* of an image |
| **Registry** | Where images are stored/shared (e.g. Docker Hub) |
| **Repo / Tag** | Repo holds image versions identified by tags (`myapi:1.0`, `myapi:latest`) |

- **Image vs container = class vs object.** One image -> many containers.
- Lifecycle: Dockerfile -> **build** -> Image -> **run** -> Container; Image -> **push/pull** -> Registry.

```
Dockerfile --build--> Image --run--> Container
                       |  ^
                  push |  | pull
                       v  |
                 Registry (Docker Hub)
```

## Use cases
1. Microservices (many small containers)
2. Moving between cloud providers (AWS, GCP)
3. Scaling across machines

## Setup
- **Docker Desktop** = Docker Engine (daemon + API + CLI) + GUI.

## Commands
```bash
docker pull <image>                         # Docker Hub -> local
docker run <image>                          # run locally
docker build -t <image_name> .              # build from Dockerfile in current dir (. = build context)
docker run -p 8888:5000 <image_name>        # port map  host_port:container_port
docker login
docker tag <image_name> <username>/<image_name>:1.0
docker push <username>/<image_name>:1.0     # upload to Docker Hub
```
- Port map: `8888` = local machine, `5000` = container -> open `localhost:8888`.
- Public repo -> anyone can pull.
- To push, the image name must start with your Docker Hub username -> `docker tag` first.

## Dockerfile (generic)
```dockerfile
FROM python:3.9                    # base image
WORKDIR /app                       # working dir in container
COPY . /app                        # copy code
RUN pip install -r requirements.txt
EXPOSE 5000                        # documents container port
CMD ["python", "app.py"]           # run command
```

## Dockerfile for the FastAPI app
```dockerfile
FROM python:3.9
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt     # cached layer
COPY . .
EXPOSE 8000
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```
```bash
docker build -t patient-api .
docker run -p 8000:8000 patient-api
```
- **`--host 0.0.0.0` is required**, otherwise the app is unreachable from outside the container.
- **Copy `requirements.txt` and `pip install` BEFORE `COPY . .`** — Docker caches layers, so code-only changes rebuild fast.
- **`EXPOSE` is documentation only**; `-p` is what actually publishes the port.

## Deployment targets (next steps)
- AWS EC2; Google Cloud Run.

## Quick interview Q&A
- **Q: Image vs container?** Blueprint vs running instance.
- **Q: What problems does Docker solve?** Consistency, isolation, scalability.
- **Q: What does `-p 8888:5000` do?** Maps host port 8888 to container port 5000.
- **Q: Does EXPOSE publish a port?** No; use `-p`.
- **Q: Why 0.0.0.0 for uvicorn in Docker?** Default 127.0.0.1 is only reachable inside the container.
- **Q: Why copy requirements first?** Layer caching speeds up rebuilds.
- **Q: What is a registry?** Storage/distribution for images (Docker Hub).
