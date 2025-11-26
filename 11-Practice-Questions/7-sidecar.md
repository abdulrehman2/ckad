A developer has created a container image acme/webapp:v1 that writes all logs into:

`/var/log/webapp/app.log`


The application does NOT write logs to stdout and cannot be modified.

Your task:

Create a Pod named webapp-pod with:

A main container:

- Name: webapp
- Image: acme/webapp:v1
- Mount shared volume at /var/log/webapp

A sidecar container:

- Name: log-sidecar
- Image: busybox
- Continuously tail the logs with:
`tail -F /logs/app.log`


Mount the same shared volume at `/logs`

A shared emptyDir volume named `log-volume`.