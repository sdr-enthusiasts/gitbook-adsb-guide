# Install Docker Compose

Compose is a tool for defining and running multi-container Docker applications. With Compose, you use a YAML file to configure your application’s services. Then, with a single command, you create, start, stop, update, etc all the services defined for your application.

Compose can be run as a container, which I highly recommend. The clever folks over at [linuxserver.io](https://www.linuxserver.io) have a great image that's easily installed.

You can simply run the following commands on your system and you should have a functional install that you can call from anywhere as `docker-compose`:

```bash
sudo curl -L --fail https://raw.githubusercontent.com/linuxserver/docker-docker-compose/master/run.sh -o /usr/local/bin/docker-compose
```

```bash
sudo chmod +x /usr/local/bin/docker-compose
```

To ensure your installation has completed successfully, you can run `docker-compose version` which will show you version information:

```text
sudo docker-compose version
```

```text
docker-compose version 1.27.4, build 4052419
docker-py version: 4.3.1
CPython version: 3.7.7
OpenSSL version: OpenSSL 1.1.1g  21 Apr 2020
```

In order to update the Compose image, you can run the following commands:

```text
docker pull linuxserver/docker-compose
docker image prune -f
```

