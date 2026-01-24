# Docker Fundamentals for DevOps (Beginner → Solid Mental Model)

> Read this gacye gacye. You are not expected to memorize.
> The goal is: “When I see a Docker command, I know *why* it exists.”

---

## 1. What Docker really is (no buzzwords)

### What it is

Docker is a tool that:

* packages an application **plus everything it needs**
* into something called an **image**
* and runs it as an isolated **container**

### The simplest mental picture

* **Image** = blueprint (like an ISO file)
* **Container** = running instance of that blueprint

Like:

* Class → Object
* Recipe → Cooked food

---

## 2. Images vs Containers (this is critical)

### Docker Image

* Read-only
* Built once
* Reusable
* Comes from:

  * Dockerfile (you build it)
  * Docker Hub (you pull it)

Example:

```bash
httpd:latest
```

### Docker Container

* A running image
* Has a lifecycle (start, stop, restart, delete)
* Can crash
* Can be destroyed safely

Example:

```bash
docker run httpd:latest
```

---

## 3. Where `docker build` comes from (the missing clue)

### What `docker build` does

It **creates an image** from a file called `Dockerfile`.

Syntax:

```bash
docker build -t IMAGE_NAME:TAG PATH
```

### Break it down slowly

```bash
docker build -t nautilus-fixed:latest .
```

| Part             | Meaning                           |
| ---------------- | --------------------------------- |
| `docker build`   | Build an image                    |
| `-t`             | Tag (give it a name)              |
| `nautilus-fixed` | Image name (any name is fine)     |
| `latest`         | Tag (version label)               |
| `.`              | Build context (current directory) |

### Why the dot (`.`) matters

Docker needs:

* the `Dockerfile`
* files referenced by `COPY` or `ADD`

So `.` means:

> “Use everything in **this folder**”

If the Dockerfile is in `/opt/docker`, you must run:

```bash
cd /opt/docker
docker build -t something .
```

---

## 4. What a Dockerfile really is

### What it is

A **step-by-step recipe** to build an image.

Docker reads it **top to bottom**.

Example structure:

```dockerfile
FROM image
RUN command
COPY file destination
CMD command
```

### Important rule

Every line creates a **layer**.
If a layer fails, the build stops.

That’s why your error showed:

```text
ERROR [2/8] RUN sed ...
```

---

## 5. Why paths matter inside Dockerfiles

This part bit you today, so let’s lock it in.

### Host paths vs Container paths

They are **not the same**.

* `/opt/docker` → exists on your VPS
* `/usr/local/apache2/conf/httpd.conf` → exists **inside the container**

When you write:

```dockerfile
RUN sed -i ... /usr/local/apache2/conf/httpd.conf
```

Docker asks:

> “Does this file exist **inside the base image**?”

If not → build fails.

---

## 6. How to check what exists inside an image (VERY IMPORTANT)

This is the superpower you just unlocked.

### Temporary inspection command

```bash
docker run --rm IMAGE command
```

Example:

```bash
docker run --rm httpd:2.4.43 ls /usr/local/apache2
```

This means:

* `--rm` → delete container after exit
* no long-running container
* safe inspection

This is how we knew:

* `conf/` exists
* `conf.d/` does not

This is **not guessing**. This is proof.

---

## 7. COPY vs RUN vs CMD (simple explanation)

### RUN

* Executes during build time
* Used to install packages or edit files

Example:

```dockerfile
RUN sed -i ...
```

### COPY

* Copies files from host → image

Example:

```dockerfile
COPY index.html /usr/local/apache2/htdocs/
```

If the file does not exist in build context → build fails.

### CMD

* Runs when the container starts
* Only one CMD per Dockerfile (last one wins)

Example:

```dockerfile
CMD ["httpd-foreground"]
```

---

## 8. Why tagging images matters

If you don’t tag:

```bash
docker build .
```

Docker creates:

```text
<none>:<none>
```

Which is annoying to use.

So we always tag:

```bash
docker build -t myimage:latest .
```

This makes it easy to:

```bash
docker run myimage:latest
```

That’s why I used `nautilus-fixed:latest`.

---

## 9. The DevOps debugging loop (memorize this)

Whenever Docker fails:

1. Read the error **line number**
2. Identify:

   * is it a path?
   * is it a command?
   * is it missing file?
3. Inspect the image if needed:

   ```bash
   docker run --rm IMAGE ls path
   ```
4. Fix only the broken part
5. Rebuild

This loop never changes.

---

## 10. Final teach-back (you should reach this soon)

> Docker builds images from Dockerfiles using the current directory as context. Errors during build usually mean a referenced file or path does not exist inside the base image. I can always inspect an image using `docker run --rm` to verify paths before fixing the Dockerfile.

When that sentence feels natural, you are **out of beginner mode**.

---
