# Local Development

When developing / testing TPO code, or analyzing TPO results it is most convenient to do it in the same runtime environment. Since all TPO are executed within a Docker container, we can 

1. Create a Docker docker for local development. The image will contain all the software and libraries used by TPO.

```
$TPO_ROOT/tpo.py -config <config.ini> boot_build -image dev
```

This will create a per-user versioned image with the following name `$USER-tpodev:$BOOT_VER`.

2. Start a Docker container based on the development image

```
$TPO_ROOT/tpodev.sh <config.ini> <container_name> [additional mounts]
```

This will start a Docker container as the same user. Additional mount-points can be provided e.g. `-v /my_data:/mnt/my_data`.

PS:
1. It's possible to detach from a container that is already running by using the `CTRL + P` and `CTRL + Q` keys. This will allow you to return to the terminal while the container continues to run in the background.
2. If you want to see a list of all running containers, you can use the `docker ps` command. To stop a running container, you can use the `docker stop` command followed by the container ID or name.
3. To attach to a running container, you can use the `docker attach` command followed by the container ID or `<container_name>` specified when started.
4. To exit the container and return to the host terminal, you can use the `CTRL + D` key combination.

