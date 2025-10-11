# ComfyUI Docker Stack

[![ComfyUI Docker](https://www.johnaldred.com/wp-content/uploads/2025/05/comfyui-docker.jpg)](https://www.johnaldred.com/running-comfyui-in-docker-on-windows-or-linux/)


Run [ComfyUI](https://github.com/comfyanonymous/ComfyUI) in Docker with Docker Compose — simple setup with persistent storage, NVIDIA GPU support, and custom node management.

This is a somewhat cleaned up version of the setup I use that works for me. I've tried to make it as easy as possible. You can install custom nodes and do updates through ComfyUI-Manager, but they will be reset once you recreate the package. This works well for me because it lets me try out custom nodes without worry of screwing up my ComfyUI setup. If it doesn't work, I can just recreate the container and reset any time I like. If I want to keep those custom nodes permanently, I just add them to entrypoint.sh so they're automatically downloaded when the container is recreated and run for the first time - the first run will take a few minutes.

It also makes it easy to update without worrying about potential conflicts. On creating (or recreating) the container, it builds a fresh image using the latest version of ComfyUI, the latest versions of the defined custom nodes and all of their dependencies. Settings, workflows, models and the output folder are persistent through recreations of the container, so you won't lose anything important or have to download models again.

## Features

- **Runs ComfyUI in Docker** with GPU (NVIDIA) support - For AMD, see below.
- **Persistent storage** for models, outputs, settings, and flows.
- **Automatic custom node installation** (see `entrypoint.sh`).
- **Simple to install and update.**

---

### AMD Support

This release is for NVIDIA GPUs. I do not have any AMD GPUs here to test with, so if you're an AMD user, check out [Colbix's AMD fork](https://github.com/Colbix/ComfyUI-Docker/tree/amd-branch).

---

## Quick Start

You can find [more complete setup instructions on my website](https://www.johnaldred.com/running-comfyui-in-docker-on-windows-or-linux/).

### **1. Prerequisites**

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) for Windows or regular [Docker](https://docs.docker.com/desktop/setup/install/linux/) for Linux.
  - On Windows: Enable WSL2 and GPU support if you want to use your NVIDIA GPU.
  - It also works through [Portainer-CE](https://hub.docker.com/r/portainer/portainer-ce), which is the way I use this setup.
- [Git](https://git-scm.com/) (to clone this repo)
- Linux users will need [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html) for NVIDIA GPU Passthrough in your Docker containers. Windows users just need the latest GPU drivers with WSL2 support.

### **2. Clone this repository**

```sh
git clone https://github.com/Kaouthia/ComfyUI-Docker.git
cd ComfyUI-Docker
```

### **3. Prepare persistent folders**

**On Windows**, create these folders (change `yourname` to your Windows username):

```
C:\Users\yourname\comfyui\models
C:\Users\yourname\comfyui\output
C:\Users\yourname\comfyui\settings
C:\Users\yourname\comfyui\flows
```

**On Linux**, create these folders (change `username` to your Linux username):
```
/home/username/comfyui/models
/home/username/comfyui/output
/home/username/comfyui/settings
/home/username/comfyui/flows
```

And update the paths in `docker-compose.yml` to match your actual folder locations.

---

### **4. Build and run the stack**

From the project directory, run:

```sh
docker compose build --no-cache && docker compose up -d
```

The first run may take several minutes as it downloads the latest version of ComfyUI and its dependencies as well as the latest versions of the custom nodes listed below, along with their dependencies.

* On completion, ComfyUI will be available at: [http://localhost:8188](http://localhost:8188)

---

## **Notes & Troubleshooting**

* **Line endings:**
  If you edit `entrypoint.sh` on Windows, ensure it uses **LF** (Unix) line endings to avoid container errors.

* **Custom nodes:**
  On first running the container, the entrypoint will automatically download several custom nodes including:
  
  - ComfyUI-Manager
  - ComfyUI_essentials
  - ComfyUI-Crystools
  - rgthree-comfy
  - ComfyUI-KJNodes
  - ComfyUI_UltimateSDUpscale

* **GPU Support:**
  This stack is set up for NVIDIA GPUs. You may need to install [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html) for GPU passthrough to work.

* **Volumes:**
  By default, your models, outputs, and other data persist in the folders you mapped on your host system. Whether you're on Windows or Linux, you'll want to modify these paths to something more suitable for your needs and folder structure.

---

## **Updating**

The cleanest way to update ComfyUI is to rebuild the image and container. This will force it to pull down the latest version of ComfyUI, along with the latest versions of the included custom nodes defined in entrypoint.sh and any dependencies.

That being said, you can also update and install custom nodes via the ComfyUI-Manager addon. Do note, however, that any custom nodes installed via ComfyUI-Manager will disappear on recreation of the container.

If you want custom nodes to persist through container recreations, add them to entrypoint.sh so that they're downloaded automatically upon rebuilding and running the container.

```
declare -A REPOS=(
  ["ComfyUI-Manager"]="https://github.com/ltdrdata/ComfyUI-Manager.git"
  ["ComfyUI_essentials"]="https://github.com/cubiq/ComfyUI_essentials.git"
  ["ComfyUI-Crystools"]="https://github.com/crystian/ComfyUI-Crystools.git"
  ["rgthree-comfy"]="https://github.com/rgthree/rgthree-comfy.git"
  ["ComfyUI-KJNodes"]="https://github.com/kijai/ComfyUI-KJNodes.git"
  ["ComfyUI_UltimateSDUpscale"]="https://github.com/ssitu/ComfyUI_UltimateSDUpscale.git"
)
```

---

## **License**

See [ComfyUI’s license](https://github.com/comfyanonymous/ComfyUI/blob/master/LICENSE) for upstream project licensing.
This repository is provided as-is for self-hosting ComfyUI in Docker.

Please note: I’m sharing this setup as-is, primarily for my own use. I do not plan to maintain prebuilt images at this time. While I appreciate interest and suggestions, I won’t be merging pull requests for automated builds or similar changes.

---

**Happy prompting!**
