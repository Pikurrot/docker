# Base-Stage
FROM opennmt/ctranslate2:latest-ubuntu20.04-cuda11.2 AS base
LABEL maintainer="Julian Reith <julianreith@gmx.de>"
LABEL description="Docker container for whisper-gui - A simple GUI made with gradio to use Whisper from https://github.com/Pikurrot/whisper-gui"

EXPOSE 7860
ENV TZ="Europe/Berlin"

# install apt prerequisites
WORKDIR /
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && \
    echo $TZ > /etc/timezone && \
    apt update && apt install -y \
        curl \
        ffmpeg \
        git \
        vim \
        wget \
        bzip2 \
        ca-certificates \
        sudo \
        pciutils \
        jq && \
    apt clean && \
    rm -rf /var/lib/apt/lists/*

# Setup Miniconda
RUN wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh && \
    bash Miniconda3-latest-Linux-x86_64.sh -b -p /opt/conda && \
    rm Miniconda3-latest-Linux-x86_64.sh && \
    /opt/conda/bin/conda init && \
    /opt/conda/bin/conda config --set always_yes yes --set changeps1 no && \
    /opt/conda/bin/conda update -q conda && \
    /opt/conda/bin/conda clean -a -y

ENV PATH="/opt/conda/bin:$PATH"

# Clone the repository
WORKDIR /whisper-gui
RUN git clone https://github.com/Pikurrot/whisper-gui .
COPY configs/config.json configs/config.json

# Replace the original main.py with your updated main.py (TEMPORARY, just for debugging, this main.py is same but with prints)
COPY main.py /whisper-gui/main.py

# Create conda environment and install CUDA dependencies
SHELL ["bash", "-c"]
RUN conda create --name whisper-gui python=3.10 -y && \
    conda clean -a -y

RUN echo "conda activate whisper-gui" >> ~/.bashrc && \
    source /opt/conda/etc/profile.d/conda.sh && \
    conda activate whisper-gui && \
    echo "NVIDIA GPU detected. Enabling CUDA support in config.json..." && \
    jq '.gpu_support = "cuda"' configs/config.json > tmp.json && mv tmp.json configs/config.json && \
    pip install torch==2.0.0 torchaudio==2.0.1 && \
    pip install git+https://github.com/m-bain/whisperx.git && \
    pip install gradio

# Prepare whisper-gui
RUN touch /VERSION && \
    latest_tag=$(git describe --tags $(git rev-list --tags --max-count=1)) && \
    echo "whisper-gui=$latest_tag" > /VERSION && \    
    mkdir -p outputs && \
    sed -i 's/demo\.launch(inbrowser=args\.autolaunch, share=args\.share)/demo.launch(server_name="0.0.0.0", server_port=7860)/' main.py && \
    chmod +x *.py scripts/*.py && \
    chmod +x *.sh scripts/*.sh

# Setup entrypoint
COPY docker-entrypoint.sh /
RUN chmod +x /docker-entrypoint.sh
ENTRYPOINT ["/docker-entrypoint.sh"]
