# Root Cause Analysis

Fill this in as you go. Bullet points are fine — be specific, not long.

---

## Defect 1: Proxy image fails to build

### Symptom / error output
* The `docker compose build` process hangs or fails during the `apt-get install -y nginx` step.

### How I diagnosed it
* I reviewed the build logs and noticed the process was stuck trying to configure Nginx because a configuration file already existed in the target directory, prompting an interactive overwrite confirmation.

### Root cause
* In `proxy/Dockerfile`, the `COPY nginx.conf` command was placed *before* the `RUN apt-get install` command. The package manager detected the manually copied file and stalled waiting for user input.

### Fix
* Moved the `COPY nginx.conf /etc/nginx/nginx.conf` instruction to appear *after* the `RUN apt-get update && apt-get install` instruction.
---

## Defect 2: Proxy cannot reach the backend

### Symptom / error output
* The `test-runner` crashed with `HTTP Error 502: Bad Gateway`. 
* Proxy logs showed: `connect() failed (111: Connection refused) ... upstream: "http://127.0.0.1:8080/api/info"`.

### How I diagnosed it
* I checked the Nginx error logs which indicated it was trying to forward requests to `127.0.0.1`.

### Root cause
* In `proxy/nginx.conf`, the upstream was hardcoded to `server 127.0.0.1:8080;`. Since Docker containers have isolated network namespaces, the proxy was looping back to itself instead of reaching the backend container.

### Fix
* Changed the upstream definition in `nginx.conf` to `server backend:8080;`, utilizing Docker Compose's built-in DNS service discovery to route traffic to the correct container IP.

---

## Defect 3: Tests still fail after fixing defect 2

### Symptom / error output
* Tests still failed with `502 Bad Gateway`.
* Backend logs showed it was still loading config keys while tests were running.

### How I diagnosed it
* Logs revealed an 8-second startup delay in `backend/app.py` (simulating config loading), while `test-runner` fired requests immediately.

### Root cause
* A race condition: `test-runner` executed requests before the `backend` was fully initialized and listening on port 8080.

### Fix, and why I chose it over the alternatives
* Added a `healthcheck` to the `backend` (`wget -qO- http://127.0.0.1:8080/health`).
* Set `proxy` to `depends_on: backend: condition: service_healthy`.
* **Why:** It is architecturally cleaner for the proxy to wait for its direct dependency (the backend). I used `127.0.0.1` instead of `localhost` in the check to avoid IPv6 resolution issues in Alpine Linux.

---

## Defect 4: CI pipeline fails

### Symptom / error output
* GitHub Actions failed with `./scripts/run-tests.sh: Permission denied` and `exit code 126`.

### How I diagnosed it
* CI logs clearly pointed to a Linux execution permission error on the runner.

### Root cause
* `scripts/run-tests.sh` lacked executable permissions (`+x`) in the Git repository metadata.

### Fix
* Ran `git update-index --chmod=+x scripts/run-tests.sh` locally to add the executable flag to the Git index, then committed and pushed.

---

## Bonus questions

Optional. Answer only the ones you got to — number them, skip the rest.

>

---

## Anything you would do differently with more time
