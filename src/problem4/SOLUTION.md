# Problem 4 Report: API Unreliability and Inaccessibility

## 1. Issues Identified
* **API Unavailability**: Requests to `/api/` endpoints were failing.
* **Missing Status Endpoint**: The `/status` endpoint was inaccessible, preventing health checks.

## 2. Diagnosis
* **Initial Check**: Verified all containers were running using `docker compose ps`.
* **Port Mismatch**: Investigated Nginx configuration (`default.conf`) and found it was proxying requests to `http://api:3001`. However, the API application was listening on port `3000`.
* **Missing Configuration**: Confirmed a `/status` route existed in the application code, but there was no corresponding `location` block in the Nginx configuration to route traffic to it.

## 3. Fixes Applied
* **Corrected Upstream Port**: Updated `nginx/conf.d/default.conf` to proxy `/api/` traffic to port `3000` instead of `3001`.
* **Exposed Status Endpoint**: Added a new `location /status` block in `nginx/conf.d/default.conf` to correctly route health check requests to the backend.

## 4. Monitoring & Alerts
To improve observability and reaction time:
* **Health Checks**: Implement an automated uptime monitor (e.g., UptimeRobot, Datadog) polling the `/status` endpoint. Alert on non-200 responses.
* **Log Aggregation**: Ingest Nginx and API logs into a centralized system (e.g., ELK Stack, Loki) to track 5xx error rates.
* **Resource Monitoring**: Monitor container CPU/Memory usage and restart counts (e.g., cAdvisor + Prometheus).

## 5. Prevention in Production
* **Configuration Management**: Use environment variables for port definitions to ensure the application and reverse proxy always share the same configuration.
* **CI/CD Integration Tests**: Add a step in the deployment pipeline that spins up the stack and runs `curl` assertions against critical endpoints (e.g., `/status`) before promoting to production.
* **Linting**: Validate Nginx configuration (`nginx -t`) during the build process to catch syntax errors early.