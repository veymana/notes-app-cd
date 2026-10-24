# notes-app-cd
Helm chart that deploys the notes app to OpenShift. Images come from Docker Hub
(built by the `notes-app` repo's `docker-publish` workflow).
## What it deploys
- `notes-api`: Deployment + Service (Spring Boot, port 8080, actuator probes), ConfigMap and Secret for DB settings
- `notes-ui`: Deployment + Service (nginx, port 8080) + edge-TLS Route
The Service names must stay `notes-api` and `notes-ui`: the UI's nginx proxies `/api/` to `http://notes-api:8080`.
PostgreSQL is not part of the chart; it expects a service reachable at `db.url`.
## Install / upgrade
```bash
helm upgrade --install notes . \
  --set dockerhubNamespace=<dockerhub-username> \
  --set api.image.tag=<tag> \
  --set ui.image.tag=<tag> \
  --set db.password='<db-password>'
oc get route notes-ui
```
Use immutable tags (`sha-<commit>` or `yymmddhh`) rather than `latest`, otherwise a new push does not trigger a rollout.
To use an existing Secret (keys `SPRING_DATASOURCE_USERNAME`, `SPRING_DATASOURCE_PASSWORD`):
`--set db.existingSecret=<name>`.
If objects with the same names already exist (e.g. from an S2I deploy), delete them first:
`oc delete all -l app=notes-api; oc delete all -l app=notes-ui`, plus the old ConfigMap, Secret and Route.
## Values
See `values.yaml`.