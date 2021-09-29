# Persistence

## Persistent Volumes with Network Storage

* Apply deployment.yaml
* Change data in folder /usr/share/nginx/html
* Restart pod and check previously changed data in folder /usr/share/nginx/html
* Apply claim.yaml
* Edit deployment.yaml, remove all comments and apply again
* Change data in folder /usr/share/nginx/html
* Restart pod and check previously changed data in folder /usr/share/nginx/html