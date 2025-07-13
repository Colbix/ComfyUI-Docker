# ComfyUI Docker Build File by John Aldred
# http://www.johnaldred.com
# http://github.com/kaouthia
#
# Updated to use AMD with ROCm by Colbix
# https://github.com/Colbix/ComfyUI-Docker

# Use a minimal Python base image (adjust version as needed)
FROM python:3.12-slim

# Allow passing in your host UID/GID (defaults 1000:1000)
ARG UID=1000
ARG GID=1000

# Install OS deps and create the non-root user
RUN apt-get update \
 && apt-get install -y --no-install-recommends git \
 && apt-get install -y wget \
 && groupadd --gid ${GID} appuser \
 && useradd --uid ${UID} --gid ${GID} --create-home --shell /bin/bash appuser \
 && rm -rf /var/lib/apt/lists/*

# The GID is not assigned to a name so we assign the GID to new group name which is used for
# /dev/kfd, /dev/dri/renderD128, and /dev/dri/renderD129
# that affect access to the main compute interface and Direct Rendering Interface
# See https://rocm.docs.amd.com/projects/install-on-linux/en/latest/how-to/docker.html#accessing-gpus-in-containers
RUN groupadd -g 992 container

# Set the working directory
WORKDIR /root

# Install Mesa/GL and GLib so OpenCV can load libGL.so.1 for ComfyUI-VideoHelperSuite
# Note: Mesa/GL hasn't been update for Ubuntu 24.04 so an older version is used
RUN apt-get update \
 && apt-get install -y --no-install-recommends libglib2.0-0 \
 && wget http://archive.ubuntu.com/ubuntu/pool/universe/m/mesa/libgl1-mesa-glx_23.0.4-0ubuntu1~22.04.1_amd64.deb \
 && apt-get install -y ./libgl1-mesa-glx_23.0.4-0ubuntu1~22.04.1_amd64.deb \
 && rm -rf /var/lib/apt/lists/*

# Copy and enable the startup script
COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh

# Switch to non-root user
USER $UID:$GID

# Install Pytorch for the supported version of Rocm, update as needed for newer versions
RUN pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/rocm6.3

# make ~/.local/bin available on the PATH so scripts like tqdm, torchrun, etc. are found
ENV PATH=/home/appuser/.local/bin:$PATH

# Set the working directory
WORKDIR /app

# Clone the ComfyUI reposiory (replace URL with the official repo)
RUN git clone https://github.com/comfyanonymous/ComfyUI.git

# Change directory to the ComfyUI folder
WORKDIR /app/ComfyUI

# Install ComfyUI dependencies
RUN pip install --no-cache-dir -r requirements.txt

# (Optional) Clean up pip cache to reduce image size
RUN pip cache purge

# Expose the port that ComfyUI will use (change if needed)
EXPOSE 8188

# Run entrypoint first, then start ComfyUI
ENTRYPOINT ["/entrypoint.sh"]
CMD ["python","/app/ComfyUI/main.py","--listen","0.0.0.0"]
