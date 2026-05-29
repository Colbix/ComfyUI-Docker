# ComfyUI Docker Build File v1.0.1 by John Aldred
# https://www.johnaldred.com
# https://github.com/kaouthia

# Use a minimal Python base image (adjust version as needed)
FROM python:3.13-slim-bookworm

# Allow passing in your host UID/GID (defaults 1000:1000)
ARG UID=1000
ARG GID=1000

# Set an environment variable path at /opt/venv
ENV VENV_PATH=/opt/venv

# Create the python virutal environment
RUN python -m venv ${VENV_PATH}

# Upgrade pip and install everything into the venv.
RUN ${VENV_PATH}/bin/pip install --upgrade pip

# Install OS deps and create the non-root user
RUN apt-get update \
 && apt-get install -y --no-install-recommends git \
 && groupadd --gid ${GID} appuser \
 && useradd --uid ${UID} --gid ${GID} --create-home --shell /bin/bash appuser \
 && chown -R ${UID}:${GID} ${VENV_PATH} \
 && rm -rf /var/lib/apt/lists/*

# The GID is not assigned to a name so we assign the GID to new group name which is used for
# /dev/kfd, /dev/dri/renderD128, and /dev/dri/renderD129
# that affect access to the Kernel Fusion Driver and Direct Rendering Interface
# See https://rocm.docs.amd.com/projects/install-on-linux/en/latest/how-to/docker.html#accessing-gpus-in-containers
RUN groupadd -g 992 render

# Install Mesa/GL and GLib so OpenCV can load libGL.so.1 for ComfyUI-VideoHelperSuite
RUN apt-get update \
 && apt-get install -y --no-install-recommends \
      libgl1 \
      libglx-mesa0 \
      libglib2.0-0 \
      libatomic1 \
      fonts-dejavu-core \
      fontconfig \
 && rm -rf /var/lib/apt/lists/*

# Copy and enable the startup script
COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh

# make ~/.local/bin available on the PATH so scripts like tqdm, torchrun, etc. are found
ENV PATH=/home/appuser/.local/bin:$PATH

# Set the working directory
WORKDIR /app

# Ensure the non-root user can write here
RUN chown -R ${UID}:${GID} /app

# Switch to non-root user
USER $UID:$GID

# Set venv’s bin directory
ENV PATH="${VENV_PATH}/bin:${PATH}"

# Install Pytorch for the supported version of Rocm, update as needed for newer versions
RUN pip install --no-cache-dir torch torchvision torchaudio --index-url https://download.pytorch.org/whl/rocm7.2

# Clone the ComfyUI repository (replace URL with the official repo)
RUN git clone https://github.com/comfyanonymous/ComfyUI.git

# Change directory to the ComfyUI folder
WORKDIR /app/ComfyUI

# Install ComfyUI dependencies
RUN pip install --no-cache-dir -r requirements.txt

# (Optional) Clean up pip cache to reduce image size
RUN pip cache purge || true

# Expose the port that ComfyUI will use (change if needed)
EXPOSE 8188

# Run entrypoint first, then start ComfyUI
ENTRYPOINT ["/entrypoint.sh"]
CMD ["python","/app/ComfyUI/main.py","--listen","0.0.0.0"]
