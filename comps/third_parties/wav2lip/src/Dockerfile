# Copyright (C) 2024 Intel Corporation
# SPDX-License-Identifier: Apache-2.0

# Use a base image
FROM python:3.11-slim

# Set this to "cpu" or "gpu" or etc
ARG ARCH="cpu"
ARG REPO_VER=n7.1.1
# Set environment variables
ENV LANG=en_US.UTF-8
ENV PYTHONPATH=/usr/local/lib/python3.11/site-packages:/home:/home/user
ENV PYTHON=/usr/bin/python3.11

# Install dependencies
RUN apt-get update -y && apt-get install -y --no-install-recommends --fix-missing \
    build-essential \
    git \
    libgl1-mesa-glx \
    libglib2.0-0 \
    libx264-dev \
    nasm \
    pkg-config \
    wget \
    yasm \
    && rm -rf /var/lib/apt/lists/*

# Install GenAIComps
RUN mkdir -p /home/user/comps
COPY comps /home/user/comps
COPY comps/third_parties/wav2lip/src/entrypoint.sh /usr/local/bin/entrypoint.sh

# Install ffmpeg with x264 software codec
RUN git clone --depth 1 --branch ${REPO_VER} https://github.com/FFmpeg/FFmpeg.git /home/user/comps/animation/src/FFmpeg && \
    cd /home/user/comps/animation/src/FFmpeg  && \
    ./configure --enable-gpl --enable-libx264 --enable-cross-compile --disable-debug --disable-doc --disable-ffprobe --disable-ffplay&& \
    make -j"$(nproc-1)" && \
    make install && \
    hash -r && chmod +x "$(which ffmpeg)" && \
    rm -rf .git 

# Upgrade pip
RUN python3 -m pip install --no-cache-dir --upgrade pip setuptools

# Install Wav2Lip from pip
RUN pip install --no-cache-dir --no-deps Wav2Lipy
RUN rm /usr/local/lib/python3.11/site-packages/Wav2Lip/__init__.py && touch /usr/local/lib/python3.11/site-packages/Wav2Lip/__init__.py
ENV PYTHONPATH="$PYTHONPATH:/usr/local/lib/python3.11/site-packages/Wav2Lip"

# Install GFPGAN from pip
RUN pip install --no-cache-dir --no-deps gfpgan
RUN touch /usr/local/lib/python3.11/site-packages/gfpgan/__init__.py
ENV PYTHONPATH="$PYTHONPATH:/usr/local/lib/python3.11/site-packages/gfpgan"

# Download pre-trained models
WORKDIR /usr/local/lib/python3.11/site-packages

# Install pip dependencies
RUN if [ ${ARCH} = "cpu" ]; then \
      pip install --no-cache-dir --extra-index-url https://download.pytorch.org/whl/cpu -r /home/user/comps/animation/src/requirements.txt; \
    else \
      pip install --no-cache-dir -r /home/user/comps/animation/src/requirements.txt; \
    fi

# Custom patches
# Modify the degradations.py file to import rgb_to_grayscale from torchvision.transforms.functional
RUN sed -i 's/from torchvision.transforms.functional_tensor import rgb_to_grayscale/from torchvision.transforms.functional import rgb_to_grayscale/' /usr/local/lib/python3.11/site-packages/basicsr/data/degradations.py

# Modify the core.py file to include 'hpu' in the device check
RUN sed -i "s/if 'cpu' not in device and 'cuda' not in device:/if 'cpu' not in device and 'cuda' not in device and 'hpu' not in device:/" /usr/local/lib/python3.11/site-packages/Wav2Lip/face_detection/detection/core.py

# To be compatible with librosa==0.10.2, instead of librosa==0.7.0 because the co-dependency numba==0.48 cannot be pip installed
RUN sed -i 's/hp.sample_rate, hp.n_fft/sr=hp.sample_rate, n_fft=hp.n_fft/' /usr/local/lib/python3.11/site-packages/Wav2Lip/audio.py

# Set the working directory
WORKDIR /home/user/comps/animation/src/

# Define the command to run when the container starts
RUN chmod +x /usr/local/bin/entrypoint.sh
ENV DEVICE="cpu"
ENTRYPOINT ["/usr/local/bin/entrypoint.sh"]
