FROM nvidia/cuda:9.0-cudnn7-runtime

SHELL ["/bin/bash", "-c"]

# 日本語化
RUN apt-get update && apt-get install -y language-pack-ja-base language-pack-ja && \
    locale-gen ja_JP.UTF-8

ENV LANG=ja_JP.UTF-8 LC_ALL=ja_JP.UTF-8

# Miniconda
ENV CONDAENV base
ENV PATH /opt/conda/bin:$PATH

RUN apt-get update --fix-missing && \
    apt-get install -y wget bzip2 ca-certificates curl git fonts-ipaexfont graphviz && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

RUN wget --quiet https://repo.anaconda.com/miniconda/Miniconda3-4.5.11-Linux-x86_64.sh -O ~/miniconda.sh && \
    /bin/bash ~/miniconda.sh -b -p /opt/conda && \
    rm ~/miniconda.sh && \
    /opt/conda/bin/conda clean -tipsy && \
    ln -s /opt/conda/etc/profile.d/conda.sh /etc/profile.d/conda.sh && \
    echo ". /opt/conda/etc/profile.d/conda.sh" >> ~/.bashrc && \
    echo 'conda activate ${CONDAENV}' >> ~/.bashrc

ENV TINI_VERSION v0.16.1
ADD https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini /usr/bin/tini
RUN chmod +x /usr/bin/tini

# パッケージインストール
ENV CONDAENV mlenv

COPY mlenv_frozen.yml ./

RUN conda update -n base conda && \
    conda env create --file mlenv_frozen.yml

COPY matplotlibrc ./
COPY notebook.json ./
COPY rcmod.py ./

RUN /opt/conda/envs/mlenv/bin/jupyter contrib nbextension install --user && \
    /opt/conda/envs/mlenv/bin/jupyter nbextensions_configurator enable --user && \
    cp notebook.json /root/.jupyter/nbconfig/ && \
    cp matplotlibrc /opt/conda/envs/mlenv/lib/python3.6/site-packages/matplotlib/mpl-data/ && \
    cp rcmod.py /opt/conda/envs/mlenv/lib/python3.6/site-packages/seaborn/

WORKDIR /src

VOLUME /src

EXPOSE 6006 8888

ENTRYPOINT ["/usr/bin/tini", "--"]

CMD /opt/conda/envs/mlenv/bin/jupyter notebook --notebook-dir=/src --ip=0.0.0.0 --port=8888 --no-browser --allow-root

