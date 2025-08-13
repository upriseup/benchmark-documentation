# 📦 How to Extract Source Code from a Google Cloud Run Docker Image

## ℹ️ Introduction

**Docker** is a platform that allows developers to package applications and all their dependencies into a standardized unit called an **image**.  
When an image is run, it becomes a **container** — a lightweight, isolated environment that executes the application exactly as intended, regardless of where it runs.

Think of it this way:
- **Image** = the recipe (code + dependencies + environment)
- **Container** = the dish made from the recipe (running instance of the image)

### In the GCP Context
- **Cloud Run** takes your application source code (or a prebuilt container image) and deploys it as a service.
- The source code is **built into a container image** using Cloud Build or another CI/CD process.
- That image is stored in **Artifact Registry** or **Container Registry**.
- When you deploy, Cloud Run starts a container **from that image** for each request.

**Example:**
If you deploy a Python service called `get--traffic--acq` to Cloud Run:
- GCP builds an image, e.g.:
  ```
  europe-west2-docker.pkg.dev/linen-waters-380615/gcf-artifacts/get--traffic--acq@sha256:f1a9...
  ```
- That image contains:
  - Your `main.py` and any other source files.
  - All installed Python packages (`requirements.txt`).
  - The base OS and Python runtime.
- Cloud Run runs containers from this image to serve requests.

By pulling that image locally, you can inspect and recover your deployed code.

---

## 🎯 Aim
To retrieve the **source code** from an older Cloud Run deployment by:
- Pulling its Docker image from Google Artifact Registry.
- Running the image locally.
- Copying files (like `main.py`) out for inspection.

---

## 🛠 Steps

### 1️⃣ Pull the Image from Artifact Registry
From the Cloud Run revision details, get the **image URL** (example below):

```bash
docker pull europe-west2-docker.pkg.dev/PROJECT_ID/gcf-artifacts/get--traffic--acq@sha256:IMAGE_DIGEST
```
This downloads the container image locally.

---

### 2️⃣ Run the Image Interactively
Start the container in interactive mode with a shell:

```bash
docker run -it IMAGE_ID /bin/bash
```
Replace `IMAGE_ID` with either the **image name** or the **ID** from `docker images`.

---

### 3️⃣ Find the Code Inside the Container
Once inside the container:
```bash
ls
cd /path/to/code
```
Look for your Python files (e.g., `main.py`).  
Common Cloud Run code locations:
- `/workspace`
- `/srv`
- `/workspace/serverless_function_source_code`

---

### 4️⃣ Get the Container ID
If you closed the interactive shell or want to copy files without attaching:
```bash
docker ps -a
```
This lists all containers. Example output:
```
CONTAINER ID   IMAGE                                             COMMAND       STATUS        NAMES
86f2eaf7521c   europe-west2-docker.pkg.dev/.../get--traffic--acq "/bin/sh"     Up 18 minutes ecstatic_torvalds
```
Use the `CONTAINER ID` in the next step.

---

### 5️⃣ Copy Files from the Container
If `main.py` is in `/workspace`, copy it locally:
```bash
docker cp 86f2eaf7521c:/workspace/main.py ./main.py
```
Or copy the whole directory:
```bash
docker cp 86f2eaf7521c:/workspace ./old_code
```

---

### 6️⃣ Inspect Locally
Now you can open the retrieved files in your editor:
```bash
code main.py  # VS Code
```
Or just view with:
```bash
cat main.py
```

---

## 💡 Notes
- **Images are large** (often ~1GB) because they include OS layers, dependencies, and sometimes unnecessary build artifacts.
- **Cloud Run** doesn’t store your "source code" directly — just the built image.
- To see different revisions, you must pull the corresponding revision’s image from Artifact Registry.
- **Containers are ephemeral** — copying files out saves them permanently to your local system.

