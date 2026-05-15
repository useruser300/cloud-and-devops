# Docker Volume

A Docker Volume is a persistent storage mechanism used to store data generated and used by Docker containers.

In simple terms, a Docker Volume is a dedicated storage location where container data is stored.

For example, if you run an application that uses a database, the application running inside the container stores its database files inside a Docker Volume.

---

# Why Do We Need Docker Volumes?

By default, data stored inside a container is tied to the container's writable layer.

If the container is removed, all data stored inside it is lost.

Docker Volumes solve this problem by storing data outside the container on the host system.

This means:

- Data persists even if the container is deleted.
- Data can be reused by new containers.
- Data can be shared between the host and containers.
- Data can be shared between multiple containers.

---

# How Docker Volumes Work

Suppose you have:

- A Host (your machine)
- A Container running an application

You mount a directory or volume from the host into the container.

The application reads and writes data inside the container path, but the actual data is stored on the host.

This is not a copy operation.

It is a direct mount.

Therefore:

- If you modify a file on the host, the changes become immediately available inside the container.
- If you modify a file inside the container, the changes become immediately available on the host.

If the container is deleted, the data remains on the host.

When you create a new container and mount the same volume, the same data becomes available again.

This is the core idea behind Docker Volumes.

---

# Types of Data Mounts in Docker

There are three main ways to mount data into containers.

---

## 1. Bind Mount (Host Path Mount)

In this method, you specify an existing directory on the host.

```bash
-v /var/mount/Dir:/var/lib/data
```

- `/var/mount/Dir` → Path on the host
- `/var/lib/data` → Path inside the container

---

## 2. Named Volume

In this method, you assign a name to the volume.

```bash
-v Jenkins_Home:/etc/default/jenkins
```

- `Jenkins_Home` → Volume name
- `/etc/default/jenkins` → Path inside the container

If you do not specify a host path, Docker automatically creates and manages the volume under:

```bash
/var/lib/docker/volumes/
```

When a new named volume is mounted into a container path that already contains data (such as Nginx's default web root), Docker copies the existing data from the image into the volume the first time it is used.

---

## 3. Anonymous Volume

In this method, you specify only the container path.

```bash
-v /var/lib/data
```

Docker automatically creates a volume with a random name and stores it under:

```bash
/var/lib/docker/volumes/
```

This type of volume is called an **Anonymous Volume**.

Anonymous volumes are typically used for temporary or automatically managed data.

---

# Practical Example Using Nginx

Suppose you have a directory named:

```text
nginx_volume
```

And you want to mount it into an Nginx container.

---

## Run a Container with a Bind Mount

```bash
docker run -d \
  --name nginx-container \
  -p 8010:80 \
  -v /full/path/to/nginx_volume:/usr/share/nginx/html \
  nginx
```

### Command Explanation

- `-d` → Run the container in detached mode
- `--name nginx-container` → Assign a name to the container
- `-p 8010:80` → Map port 8010 on the host to port 80 in the container
- `-v` → Mount the volume
- `nginx` → Docker image name

---

## Why Is the Page Empty?

Because the mounted directory is empty.

Nginx serves files from the following default directory:

```bash
/usr/share/nginx/html
```

---

## Access the Container

```bash
docker exec -it nginx-container /bin/bash
```

---

## Set Permissions on the Host Directory

If you cannot create files in the directory:

```bash
sudo chown -R $USER:$USER nginx_volume
```
or 

```bash
chmod -R 755 nginx_volume
```

---

## Create an `index.html` File

On the host:

```bash
echo "<h1>Hello Docker Volume</h1>" > index.html
```

Open your browser and navigate to:

```text
http://localhost:8010
```

The page will display the content immediately.

---

## Live Synchronization

Any file change on the host becomes immediately available inside the container.

Any file change inside the container becomes immediately available on the host.

---

# Data Persistence After Container Removal

Stop and remove the container:

```bash
docker stop nginx-container
docker rm nginx-container
```

Run it again using the same volume:

```bash
docker run -d \
  --name nginx-container \
  -p 8010:80 \
  -v /full/path/to/nginx_volume:/usr/share/nginx/html \
  nginx
```

The same data appears again because it is stored outside the container.

---

# Using a Named Volume

```bash
docker run -d \
  --name nginx-container \
  -p 8010:80 \
  -v nginx-volume:/usr/share/nginx/html \
  nginx
```

---

## List Docker Volumes

```bash
docker volume ls
```

---

## Reuse the Same Named Volume

You can remove the container and create a new one using the same `nginx-volume`.

The stored data remains intact.

---

# Modify Data from Inside the Container

```bash
docker exec -it nginx-container /bin/bash
cd /usr/share/nginx/html
vim index.html
```

Any changes are saved to the volume and are also reflected on the host.

---

# Anonymous Volume Example

```bash
docker run -d \
  --name nginx-container \
  -p 8010:80 \
  -v /usr/share/nginx/html \
  nginx
```

Docker creates a volume with a random name.

---

## Find the Anonymous Volume Name

```bash
docker volume ls
```

A randomly generated volume name will appear.

---

## Reuse the Anonymous Volume

Copy the generated volume name and use it like a named volume:

```bash
docker run -d \
  --name nginx-container \
  -p 8010:80 \
  -v <volume-name>:/usr/share/nginx/html \
  nginx
```

---

# Remove a Docker Volume

If the volume is currently in use, stop and remove the container first:

```bash
docker stop nginx-container
docker rm nginx-container
```

Then remove the volume:

```bash
docker volume rm nginx-volume
```

---

## Verify Deletion

```bash
docker volume ls
```

---

# Summary

Docker Volumes provide persistent storage for container data.

Data remains available even after containers are deleted.

---

## Main Types of Data Mounts in Docker

1. Bind Mount (Host Path)
2. Named Volume
3. Anonymous Volume

---

## Benefits of Docker Volumes

- Persistent data storage
- Data sharing between host and containers
- Data reuse across multiple containers
- Easy backup and restore
- Ideal for databases and stateful applications

---

## Common Applications That Use Docker Volumes

- Nginx
- Jenkins
- Grafana
- Prometheus
- Portainer
- MySQL
- PostgreSQL
- MongoDB
