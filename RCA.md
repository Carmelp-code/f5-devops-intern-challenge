# Root Cause Analysis

Fill this in as you go. Bullet points are fine — be specific, not long.

## Defect 1: Proxy image fails to build

### Symptom / error output
nginx.conf (Y/I/N/O/D/Z) [default=N]?
dpkg: error processing package nginx-common (--configure):
end of file on stdin at conffile prompt

### How I diagnosed it
Ran ./scripts/run-tests.sh and examined the build error. ChatGPT suggested reversing the order of COPY and RUN. I researched what these instructions mean and why their order matters. 

### Root cause
The Dockerfile proxy image copied a custom nginx configuration before installing Nginx. During installation, dpkg asked which configuration version to keep - the existing version or the package maintainer's version. Docker builds are non-interactive, and apt-get -y doesnt resolve this configuration conflict. So standard input reaches the end of file EOF. so nginx-common failed to configure, and then nginx failed because its dependency was left unconfigures.

### Fix
Install Nginx first, then copied the custom configuration over the default. Verified by successfully rebuilding the image.


## Defect 2: Proxy cannot reach the backend

### Symptom / error output
connect() failed (111: Connection refused)
upstream: "http://127.0.0.1:8080/api/info"
HTTP Error 502: Bad Gateway

### How I diagnosed it
Examined the Nginx logs in Docker Desktop and compared the upstream address in nginx.conf with the services defined in docker-compose.yml.

### Root cause

the upstream address in the proxy configuration was 127.0.0.1:8080. Inside a container, 127.0.0.1 refers to the  container itself, not the backend container. Nginx was connecting to the wrong place.

### Fix
Changed the upstream from 127.0.0.1:8080 to backend:8080. Docker Compose allows containers to communicate using service names.
Verified that the proxy could reach the backend once it was running.

---

## Defect 3: Tests still fail after fixing defect 2

### Symptom / error output

upstream: "http://172.18.0.2:8080/api/info"
connect() failed (111: Connection refused)
TEST RUNNER CRASHED: HTTP Error 502


### How I diagnosed it
Ran the integration tests again. The github pipeline showed that Nginx was resolving the backend container correctly, but the connection was still refused. Running the services separately worked once the backend had started.

### Root cause
Nginx tried to contact the backend before it was ready (the request arrived before the backend was listening).
the compose file has: 
    depends_on:
    - backend
which starts the backend first but doesn't wait until it's ready. 

### Fix, and why I chose it over the alternatives
I added a backend healthcheck in docker-compose.yml that requests /health every 2 seconds, and changed the proxy dependency to:

depends_on:
  backend:
    condition: service_healthy

Now Docker Compose waits for the backend to become healthy before starting the proxy.

I chose a Docker health check over Python retries. Both could solve the timing problem, but a health check handles readiness in Docker Compose, where the services are started and managed. This keeps the Python tests focused on testing the application rather than waiting for its services to start.

## Defect 4: CI pipeline fails

### Symptom / error output
GitHub Actions failed at: 
./scripts/run-tests.sh: Permission denied
Error: Process completed with exit code 126

### How I diagnosed it
The Docker tests passed locally, but GitHub Actions failed before Docker started. I checked the failed workflow logs and saw "permission denied".
I checked permissions by using: "git ls-files --stage scripts/run-tests.sh"-  and checked the script's Git file mode, which was 100644 (not executable).



### Root cause
GitHub Actions runs on Linux, where a script needs executable permission to run directly. The script was missing that permission in Git. This can be overlooked when working on Windows.


### Fix
I marked the script executable in Git:
git update-index --chmod=+x run-tests.sh
This changed the file mode from 100644 to 100755. The Docker tests had already passed locally, and the workflow then passed the script to GitHub as executable.


---

## Bonus questions

## 1. Image Size

I replaced ubuntu:24.04 with nginx:alpine, which is a smaller image that already includes Nginx.

This allowed me to remove the entire apt-get installation block, including unnecessary production tools like vim, git, and htop.

## 2. Security: root vs. non-root
I changed the Dockerfile to run Nginx as USER nginx instead of root to limit its permissions.
This required a few configuration changes: 

## Listening port: 80 -> 8080
Port 80 normally requires extra privileges. Since I wanted Nginx to run without root, I changed the port it receives requests on to 8080.

## Docker Compose: 8081:8080
I changed the Docker Compose port mapping so requests arriving at port 8081 on my computer would be forwarded to Nginx's new port, 8080.

## pid /tmp/nginx.pid
Nginx needs to write a file containing its process ID. The non-root user may not have permission to write it in the default location, so I moved it to /tmp, which is normally writable.

## Cache directory permissions
Nginx needs a directory to store temporary files, but the non-root user didn't have permission to write there. After switching to non-root, I encountered this error: 
        "mkdir() "/var/cache/nginx/client_temp" failed (13: Permission denied)"
I changed the directory's ownership to the Nginx user:
        RUN mkdir -p /var/cache/nginx && \
            chown -R nginx:nginx /var/cache/nginx


## Anything you would do differently with more time
I would spend more time learning Docker, Nginx, and CI/CD so I could better understand the system and design additional tests, rather than just fixing the failures I encountered.
