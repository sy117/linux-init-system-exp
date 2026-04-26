# Linux Init System Experiment

This project demonstrates and compares two different init systems running in Docker containers: **s6-overlay** and **systemd**. Both implementations provide the same services (SSH and an Echo HTTP server) to highlight the differences and similarities between the two init systems.

## Project Structure

```
.
├── ubuntu-s6-demo/          # s6-overlay init system demo
│   ├── Dockerfile
│   ├── echo_server.py
│   └── s6-overlay/
│       └── s6-rc.d/
│           ├── echo-server/
│           ├── sshd/
│           └── user/
├── ubuntu-systemd-demo/     # systemd init system demo
│   ├── Dockerfile
│   ├── echo_server.py
│   └── systemd/
│       ├── echo-server.service
│       └── sshd.service
└── README.md (this file)
```

## Services

Both containers run the same two services:

1. **SSH Server** - OpenSSH server listening on port 22
   - Default credentials: `root:root` (for testing purposes only)
   
2. **Echo Server** - Python HTTP server listening on port 8000
   - Simple request echo service that responds with the requested path

## Prerequisites

- Docker installed and running
- Basic command-line knowledge
- `ssh` client (pre-installed on macOS/Linux)
- `curl` or web browser for testing HTTP endpoints

## ubuntu-s6-demo (s6-overlay)

### Build the s6-overlay Image

```bash
cd ubuntu-s6-demo
docker build -t ubuntu-s6-demo .
```

### Run the s6-overlay Container

```bash
docker run -p 2222:22 -p 8000:8000 ubuntu-s6-demo
```

> Note: this command does not assign a container name. If you want a fixed name for later commands, add `--name s6-demo`.

### Test s6-overlay Services

#### Test SSH Connection

```bash
ssh -o StrictHostKeyChecking=no root@127.0.0.1 -p 2222
# Enter password: root
```

After connecting, verify the services are running:

```bash
# Inside container (s6-overlay)
ps aux | grep -E "sshd|python3|s6"
```

#### Test Echo Server

```bash
curl http://localhost:8000/hello
curl http://localhost:8000/test/path
```

Expected output:
```
Echo: /hello
Echo: /test/path
```

## ubuntu-systemd-demo (systemd)

### Build the systemd Image

```bash
cd ubuntu-systemd-demo
docker build -t ubuntu-systemd-demo .
```

### Run the systemd Container

**IMPORTANT: systemd requires privileged mode to run properly.**

```bash
docker run -d \
  --name systemd-demo \
  --privileged \
  -p 2223:22 \
  -p 8001:8000 \
  ubuntu-systemd-demo
```

**Note:** The `--privileged` flag is required for systemd to function correctly inside the container. Without it, systemd services may fail to start properly.

### Test systemd Services

#### Test SSH Connection

```bash
ssh -o StrictHostKeyChecking=no root@127.0.0.1 -p 2223
# Enter password: root
```

After connecting, verify services:

```bash
# Inside container (systemd)
systemctl status
systemctl list-units --type=service
systemctl status echo-server
systemctl status ssh
```

#### Test Echo Server

```bash
curl http://localhost:8001/hello
curl http://localhost:8001/test/path
```

Expected output:
```
Echo: /hello
Echo: /test/path
```

## Running Both Demos Simultaneously

If you want to run both containers at the same time, use different port mappings to avoid conflicts:

### Run s6-overlay Container (with different ports)

```bash
docker run -p 2222:22 -p 8000:8000 ubuntu-s6-demo
```

### Run systemd Container (with different ports)

```bash
docker run -d \
  --name systemd-demo \
  --privileged \
  -p 2223:22 \
  -p 8001:8000 \
  ubuntu-systemd-demo
```

### Test Both Containers

```bash
#!/bin/bash

echo "===== Testing s6-overlay Demo ====="
echo "Testing SSH..."
sshpass -p root ssh -o StrictHostKeyChecking=no root@127.0.0.1 -p 2222 "ps aux | grep -E 'sshd|python3'"

echo -e "\nTesting Echo Server..."
curl http://localhost:8000/s6-test
curl http://localhost:8000/init-systems

echo -e "\n\n===== Testing systemd Demo ====="
echo "Testing SSH..."
sshpass -p root ssh -o StrictHostKeyChecking=no root@127.0.0.1 -p 2223 "systemctl status"

echo -e "\nTesting Echo Server..."
curl http://localhost:8001/systemd-test
curl http://localhost:8001/init-systems
```

## Container Logs and Inspection

```bash
# View container logs
docker logs s6-demo
docker logs systemd-demo

# Execute commands inside running container
docker exec -it s6-demo ps aux
docker exec -it systemd-demo systemctl list-units

# Inspect container details
docker inspect s6-demo
docker inspect systemd-demo
```

## Cleaning Up

```bash
# Stop containers
docker stop s6-demo systemd-demo

# Remove containers
docker rm ubuntu-s6-demo systemd-demo

# Remove images
docker rmi ubuntu-s6-demo ubuntu-systemd-demo
```

## Key Differences Between Init Systems

### s6-overlay
- Lightweight, composable init system
- Service supervision through `s6-rc`
- Minimal footprint
- Services defined in `s6-rc.d/` directories
- Graceful shutdown handling

### systemd
- Full-featured init system
- Service management through unit files
- Rich feature set and ecosystem
- Services defined in `/etc/systemd/system/`
- Integrated logging, dependency management, and resource limits

## Troubleshooting

### Port Already in Use

If ports are already in use, modify the port mappings:

```bash
docker run -d -p 3333:22 -p 9000:8000 --name s6-demo linux-init-s6:latest
```

### SSH Connection Refused

Ensure the container is running:

```bash
docker ps | grep -E "s6-demo|systemd-demo"
```

### Cannot Access Echo Server

Verify the container is listening on port 8000:

```bash
docker exec s6-demo netstat -tlnp | grep 8000
docker exec systemd-demo netstat -tlnp | grep 8000
```

### Service Not Starting

Check container logs for errors:

```bash
docker logs --follow s6-demo
docker logs --follow systemd-demo
```

## Additional Resources

- [s6-overlay Documentation](https://github.com/just-containers/s6-overlay)
- [systemd Official Documentation](https://systemd.io/)
- [Docker Documentation](https://docs.docker.com/)

## Notes

- Default credentials (`root:root`) are for testing purposes only and should not be used in production
- Both containers disable PAM authentication for simplicity in testing
- The echo server is not a production-grade HTTP server; it's intended for demonstration purposes only
