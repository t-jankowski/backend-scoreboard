##### SCOREBOARD DOCKERFILE TEMPLATE #####
FROM ubuntu:18.04

# Disable interactive installation mode
ENV DEBIAN_FRONTEND=noninteractive

# Install Python dependencies
RUN apt-get update && apt-get install -y \
    python3 \
    python3-dev \
    python3-pip && \
    apt-get clean autoclean && apt-get autoremove -y

RUN pip3 install --upgrade pip setuptools wheel

# Copy local directories
COPY ./test /root/test
COPY ./setup /root/setup

# Install test report dependencies
RUN pip3 install --no-cache-dir -r /root/setup/requirements_report.txt

############## ONNX Backend dependencies ###########
ENV ONNX_BACKEND="test_onnx.utils.onnx_backend"

ARG OV_DIST_DIR=/openvino/dist

RUN apt-get install -y --no-install-recommends \
    git \
    wget \
    build-essential && \
    apt-get clean autoclean && apt-get autoremove -y
RUN pip3 install --upgrade cmake cython

# Build OpenVINO
WORKDIR /
RUN git clone --recurse-submodules --depth 1 -q https://github.com/openvinotoolkit/openvino.git
WORKDIR /openvino/build
RUN cmake .. \
    -DENABLE_CLDNN=OFF \
    -DENABLE_GNA=OFF \
    -DENABLE_OPENCV=OFF \
    -DENABLE_VPU=OFF \
    -DNGRAPH_DEBUG_ENABLE=OFF \
    -DENABLE_TESTS=OFF \
    -DENABLE_BEH_TESTS=OFF \
    -DENABLE_FUNCTIONAL_TESTS=OFF \
    -DENABLE_PROFILING_ITT=OFF \
    -DENABLE_SAMPLES=OFF \
    -DENABLE_SPEECH_DEMO=OFF \
    -DENABLE_MKL_DNN=ON \
    -DENABLE_PYTHON=ON \
    -DNGRAPH_PYTHON_BUILD_ENABLE=ON \
    -DPYTHON_EXECUTABLE=/usr/bin/python3 \
    -DNGRAPH_ONNX_IMPORT_ENABLE=ON \
    -DNGRAPH_INTERPRETER_ENABLE=ON \
    -DNGRAPH_DYNAMIC_COMPONENTS_ENABLE=ON \
    -DCMAKE_INSTALL_PREFIX=${OV_DIST_DIR}
RUN make -j $(nproc) install

# RUN Install ngraph wheel
WORKDIR /openvino/ngraph/python
RUN git clone --branch v2.5.0 --depth 1 -q https://github.com/pybind/pybind11.git
ENV NGRAPH_CPP_BUILD_PATH=${OV_DIST_DIR}
ENV LD_LIBRARY_PATH=${OV_DIST_DIR}/lib
ENV DYLD_LIBRARY_PATH=${DYLD_LIBRARY_PATH}:${OV_DIST_DIR}/lib
ENV PYTHONPATH=/openvino/bin/intel64/Release/lib:/openvino/bin/intel64/Release/lib/python_api/python3.8:/openvino/ngraph/python/tests/test_onnx/utils:/openvino/ngraph/python/tests:/openvino/ngraph/python:${PYTHONPATH}
RUN pip3 install -r requirements.txt
RUN pip3 install -r requirements_test.txt
RUN python3 setup.py bdist_wheel
WORKDIR /openvino/ngraph/python/dist
# TODO avoid ls
RUN pip3 install `ls`

# TODO Problem is that setupvars.sh sets export variables not environmental,
#      thus these variables disappear once shell exits - workaround below.
# IF NO PROBLEM
# RUN /bin/bash -c "source ${OV_DIST_DIR}/bin/setupvars.sh"
# ELSE WORKAROUND
ENV PYTHONPATH=${OV_DIST_DIR}/python/python3.6:${OV_DIST_DIR}/python/python3:${OV_DIST_DIR}/deployment_tools/model_optimizer:/openvino/bin/intel64/Release/lib:/openvino/bin/intel64/Release/lib/python_api/python3.8:/openvino/ngraph/python/tests/test_onnx/utils:/openvino/ngraph/python/tests:/openvino/ngraph/python:
ENV INTEL_OPENVINO_DIR=${OV_DIST_DIR}
ENV PATH=${OV_DIST_DIR}/deployment_tools/model_optimizer:${OV_DIST_DIR}/deployment_tools/model_optimizer:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
ENV HDDL_INSTALL_DIR=${OV_DIST_DIR}/deployment_tools/inference_engine/external/hddl
ENV LD_LIBRARY_PATH=${OV_DIST_DIR}/deployment_tools/inference_engine/external/hddl/lib:${OV_DIST_DIR}/deployment_tools/inference_engine/external/gna/lib:${OV_DIST_DIR}/deployment_tools/inference_engine/external/mkltiny_lnx/lib:${OV_DIST_DIR}/deployment_tools/inference_engine/external/tbb/lib:${OV_DIST_DIR}/deployment_tools/inference_engine/lib/intel64:${OV_DIST_DIR}/lib
ENV INTEL_CVSDK_DIR=${OV_DIST_DIR}
ENV InferenceEngine_DIR=${OV_DIST_DIR}/deployment_tools/inference_engine/share
# END WORKAROUND

# Install dependencies
RUN pip3 install onnx

WORKDIR /
####################################################

CMD . /root/setup/docker-setup.sh && \
    pytest /root/test/test_backend.py --onnx_backend=${ONNX_BACKEND} -k 'not _cuda' -v
