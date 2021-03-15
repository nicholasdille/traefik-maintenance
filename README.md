# traefik-maintenance

This repository demonstrated how to improve running services behind traefik:

1. A catch-all service to replace the default 404 page
2. Display custom error pages for server errors
3. Show a service only when a cookie is set
4. Show a message of the day before access to the service is granted

I have published two posts explaining the contents of this directory:

- [Using #traefik error pages to handle unavailable services](https://dille.name/blog/2021/03/14/using-traefik-error-pages-to-handle-unavailable-services/)
- [Using #traefik to display maintenance information](https://dille.name/blog/2021/03/14/using-traefik-to-display-maintenance-information/)

All compose files have been written according to the [compose spec](https://github.com/compose-spec/compose-spec/blob/master/spec.md) and have been tested with [compose-cli](https://github.com/docker/compose-cli). Files have not been renamed to `compose.yaml` for compatability with `docker-compose`.
