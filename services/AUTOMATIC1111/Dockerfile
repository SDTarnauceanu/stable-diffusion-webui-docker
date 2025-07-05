FROM alpine/git:2.36.2 as download

COPY clone.sh /clone.sh

RUN . /clone.sh stable-diffusion-webui-assets https://github.com/AUTOMATIC1111/stable-diffusion-webui-assets.git 6f7db241d2f8ba7457bac5ca9753331f0c266917

RUN . /clone.sh stable-diffusion-stability-ai https://github.com/Stability-AI/stablediffusion.git f16630a927e00098b524d687640719e4eb469b76 \
  && rm -rf assets data/**/*.png data/**/*.jpg data/**/*.gif

RUN . /clone.sh BLIP https://github.com/salesforce/BLIP.git 3a29b7410476bf5f2ba0955827390eb6ea1f4f9d
RUN . /clone.sh k-diffusion https://github.com/crowsonkb/k-diffusion.git 8018de0b43da8d66617f3ef10d3f2a41c1d78836
RUN . /clone.sh clip-interrogator https://github.com/pharmapsychotic/clip-interrogator bc07ce62c179d3aab3053a623d96a071101d11cb
RUN . /clone.sh generative-models https://github.com/Stability-AI/generative-models 0ad7de9a5cb53fd63d6d30a4f385485e72e08597
RUN . /clone.sh stable-diffusion-webui-assets https://github.com/AUTOMATIC1111/stable-diffusion-webui-assets 6f7db241d2f8ba7457bac5ca9753331f0c266917

FROM pytorch/pytorch:2.7.1-cuda12.6-cudnn9-runtime

ENV DEBIAN_FRONTEND=noninteractive PIP_PREFER_BINARY=1

RUN --mount=type=cache,target=/var/cache/apt \
  apt-get update && \
  apt-get install -y fonts-dejavu-core rsync git jq moreutils aria2 \
  ffmpeg libglfw3-dev libgles2-mesa-dev pkg-config libcairo2 libcairo2-dev build-essential

WORKDIR /
RUN --mount=type=cache,target=/root/.cache/pip \
  git clone https://github.com/AUTOMATIC1111/stable-diffusion-webui.git && \
  cd stable-diffusion-webui && \
  git reset --hard v1.10.1 && \
  pip install -r requirements_versions.txt

# Patch gradio after it’s installed
RUN sed -i 's/in_app_dir = .*/in_app_dir = True/g' $(python -c "import gradio; print(gradio.__file__.replace('__init__.py','routes.py'))")

ENV ROOT=/stable-diffusion-webui

COPY --from=download /repositories/ ${ROOT}/repositories/
RUN mkdir ${ROOT}/interrogate && cp ${ROOT}/repositories/clip-interrogator/clip_interrogator/data/* ${ROOT}/interrogate

RUN --mount=type=cache,target=/root/.cache/pip \
  pip install pyngrok xformers==0.0.31 dctorch==0.1.2 \
  git+https://github.com/TencentARC/GFPGAN.git@7552a7791caad982045a7bbe5634bbf1cd5c8679 \
  git+https://github.com/openai/CLIP.git@dcba3cb2e2827b402d2701e7e1c7d9fed8a20ef1 \
  git+https://github.com/mlfoundations/open_clip.git@v2.32.0

RUN apt-get -y install libgoogle-perftools-dev && apt-get clean
ENV LD_PRELOAD=libtcmalloc.so

COPY . /docker

RUN git config --global --add safe.directory '*'

WORKDIR ${ROOT}
ENV NVIDIA_VISIBLE_DEVICES=all
ENV CLI_ARGS=""
EXPOSE 7860
ENTRYPOINT ["/docker/entrypoint.sh"]
CMD python -u webui.py --listen --port 7860 ${CLI_ARGS}
