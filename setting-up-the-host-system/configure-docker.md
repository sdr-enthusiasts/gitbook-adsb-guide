# Configure Docker

Before starting any containers, you'll want to configure docker container log rotation. This is important, as without log rotation, each container's log will continue to grow in size until it consumes all the disk space available to `/var/lib/docker/containers`.

It should be noted that on a fresh docker install, the file `/etc/docker/daemon.json` will likely not exist. If this is the case, just create a new file at that path and populate it with the following:

```json
{
  "log-driver": "local",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
```

After making the above change, you'll need to restart docker:

```bash
service docker restart
```
