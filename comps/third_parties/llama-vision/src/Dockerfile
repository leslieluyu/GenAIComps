# Copyright (C) 2024 Intel Corporation
# SPDX-License-Identifier: Apache-2.0

# HABANA environment
FROM vault.habana.ai/gaudi-docker/1.20.1/ubuntu22.04/habanalabs/pytorch-installer-2.6.0 AS hpu

ENV LANG=en_US.UTF-8

RUN apt-get update && apt-get install -y --no-install-recommends --fix-missing \
    curl \
    git-lfs \
    libgl1-mesa-glx \
    libjemalloc-dev

RUN useradd -m -s /bin/bash user && \
    mkdir -p /home/user && \
    chown -R user /home/user/

RUN git lfs install

COPY comps /home/user/comps

WORKDIR /home/user/comps/third_parties/llama-vision/src

RUN pip install --no-cache-dir --upgrade pip setuptools && \
    pip install --no-cache-dir -r requirements.txt && \
    pip install --no-cache-dir --upgrade Pillow

RUN pip install --no-cache-dir git+https://github.com/huggingface/optimum-habana@v1.17.0

ENV PYTHONPATH=/root:/home/user

USER user

WORKDIR /home/user/comps/third_parties/llama-vision/src

ENTRYPOINT ["python", "lvm.py"]
