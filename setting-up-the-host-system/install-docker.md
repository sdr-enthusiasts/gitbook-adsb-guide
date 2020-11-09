# Install Docker

In short, you should be able to enter the following commands at your Linux shell, which will automate the docker installation process:

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

After the above installation script finishes running, you should be able to test your docker installation by issuing the command:

```text
sudo docker run --rm hello-world
```

You should be presented with the following output:

```text
Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

If you get stuck, you can try following the instructions located here: [https://docs.docker.com/install/](https://docs.docker.com/install/)

