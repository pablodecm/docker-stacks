# Copyright (c) Jupyter Development Team.
# Distributed under the terms of the Modified BSD License.
ARG BASE_CONTAINER=treelogic/tensorflow-notebook-gpu


# Install OpenVino
FROM $BASE_CONTAINER as openvino_builder

USER root

# install OpenVINO
ARG D_OPEN_VINO=http://registrationcenter-download.intel.com/akdlm/irc_nas/15944/l_openvino_toolkit_p_2019.3.334.tgz
RUN apt-get update && \
    apt-get install -y cpio \
    && rm -rf /var/lib/apt/lists/*

RUN cd /tmp && \
    wget -c $D_OPEN_VINO && \
    tar xf l_openvino_toolkit_p_2019.3.334.tgz && \
    cd l_openvino_toolkit_p_2019.3.334 && \
    sed -i 's/decline/accept/g' silent.cfg && \
    ./install.sh -s silent.cfg && \
    cd /opt/intel/openvino/install_dependencies && \
    ./install_openvino_dependencies.sh && \
    echo "source /opt/intel/openvino/bin/setupvars.sh" >> $HOME/.bashrc

USER $NB_UID

FROM $BASE_CONTAINER

# copy and setup OpenVino
ENV PYTHONPATH "/opt/intel/openvino_2019.3.334/python/python3.6:/opt/intel/openvino_2019.3.334/python/python3:/opt/intel/openvino_2019.3.334/deployment_tools/open_model_zoo/tools/accuracy_checker:/opt/intel/openvino_2019.3.334/deployment_tools/model_optimizer:${PYTHONPATH}"
ENV LD_LIBRARY_PATH "/opt/intel/openvino_2019.3.334/opencv/lib:/opt/intel/opencl:/opt/intel/openvino_2019.3.334/deployment_tools/inference_engine/external/hddl/lib:/opt/intel/openvino_2019.3.334/deployment_tools/inference_engine/external/gna/lib:/opt/intel/openvino_2019.3.334/deployment_tools/inference_engine/external/mkltiny_lnx/lib:/opt/intel/openvino_2019.3.334/deployment_tools/inference_engine/external/tbb/lib:/opt/intel/openvino_2019.3.334/deployment_tools/inference_engine/lib/intel64:/opt/intel/openvino_2019.3.334/openvx/lib:/usr/local/nvidia/lib:/usr/local/nvidia/lib64:${LD_LIBRARY_PATH}"
COPY --from=openvino_builder /opt/intel/ /opt/intel/

USER root

RUN  cd /opt/intel/openvino/install_dependencies && \
     ./install_openvino_dependencies.sh && \
     echo "source /opt/intel/openvino/bin/setupvars.sh" >> /home/$NB_USER/.bashrc

USER $NB_UID

LABEL maintainer="Jupyter Project <jupyter@googlegroups.com>"

# Install torch and related libraries
RUN pip install --quiet \
    'torch==1.3.1' \
    'torchvision==0.4.2' \
    'fastai==1.0.60' \
    'pytorch-lightning==0.6.0' && \
    fix-permissions $CONDA_DIR && \
    fix-permissions /home/$NB_USER

RUN fix-permissions /opt/intel

ENV TF_FORCE_GPU_ALLOW_GROWTH "true"