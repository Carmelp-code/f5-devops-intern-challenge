# Root Cause Analysis

Fill this in as you go. Bullet points are fine — be specific, not long.

---

## Defect 1: Proxy image fails to build

### Symptom / error output
#24 21.28 Setting up nginx-common (1.24.0-2ubuntu7.18) ...
#24 21.32 Configuration file '/etc/nginx/nginx.conf'
#24 21.32  ==> File on system created by you or by a script.
#24 21.32  ==> File also in package provided by package maintainer.
#24 21.32    What would you like to do about it ?  Your options are:
#24 21.32     Y or I  : install the package maintainer's version
#24 21.32     N or O  : keep your currently-installed version
#24 21.32       D     : show the differences between the versions
#24 21.32       Z     : start a shell to examine the situation
#24 21.32  The default action is to keep your current version.
#24 21.32 *** nginx.conf (Y/I/N/O/D/Z) [default=N] ? dpkg: error processing package nginx-common(--configure):
#24 21.32  end of file on stdin at conffile prompt

[...]

#24 22.31 dpkg: dependency problems prevent configuration of nginx:
#24 22.31  nginx depends on nginx-common (= 1.24.0-2ubuntu7.18); however:
#24 22.31   Package nginx-common is not configured yet.
#24 22.31 dpkg: error processing package nginx (--configure):
#24 22.31  dependency problems - leaving unconfigured

[...]

#24 24.97 Errors were encountered while processing:
#24 24.97  nginx-common
#24 24.97  nginx
#24 24.99 E: Sub-process /usr/bin/dpkg returned an error code (1)
#24 ERROR: process "/bin/sh -c apt-get update &&     apt-get install -y nginx curl htop vim git" did not complete successfully: exit code: 100

### How I diagnosed it

### Root cause
The proxy image already containt a configuration (/etc/nginx/nginx.conf) when nginx-common was installed. dpkg asks which version to keep - the existing version or the package maintainer' version. Docker builds are non-interactive - nobody types N, so standard input reaches the end of file EOF. so nginx-common failed to configure, and then nginx failed because its dependency was left unconfigures. 

### Fix
Install nginx, copy the custom nginx.conf afterward. That avoids package-install conflict. 
---

## Defect 2: Proxy cannot reach the backend

### Symptom / error output
proxy-1        | 2026/10/12 08:03:00 [error] 6#6: *2 connect() failed (111: Connection refused) while connecting to upstream, client: 172.18.0.4, server: localhost, request: "GET /api/info HTTP/1.1", upstream: "http://127.0.0.1:8080/api/info", host: "proxy:80"
### How I diagnosed it

### Root cause

the upstream address in the proxy configuration is 127.0.0.1:8080

### Fix

---

## Defect 3: Tests still fail after fixing defect 2

### Symptom / error output

test-runner-1  | Testing proxied backend /api/info...


proxy-1        | 2026/10/12 08:09:12 [error] 7#7: *2 connect() failed (111: Connection refused) while connecting to upstream, client: 172.18.0.4, server: localhost, request: "GET /api/info HTTP/1.1", upstream: "http://172.18.0.2:8080/api/info", host: "proxy:80"


test-runner-1  | TEST RUNNER CRASHED: HTTP Error 502: Bad Gateway


### How I diagnosed it

### Root cause

### Fix, and why I chose it over the alternatives

---

## Defect 4: CI pipeline fails

### Symptom / error output

### How I diagnosed it

### Root cause

### Fix

---

## Bonus questions

Optional. Answer only the ones you got to — number them, skip the rest.

>

---

## Anything you would do differently with more time
