FROM --platform=${TARGETPLATFORM:-linux/amd64} ghcr.io/openfaas/classic-watchdog:0.2.0 as watchdog
FROM python:3.7

FROM ubuntu:20.04

RUN apt-get update && apt-get install -y software-properties-common
RUN add-apt-repository ppa:deadsnakes/ppa
RUN apt-get update
RUN apt-get install -y build-essential libpq-dev libxml2-dev libxslt1-dev libldap2-dev libsasl2-dev libffi-dev curl pkg-config unzip wget zlib1g libatlas-base-dev python3.7-dev python3.7-distutils libhdf5-103 libhdf5-dev python3-testresources
RUN apt-get update
RUN apt-get install -y python3-numpy libblas-dev liblapack-dev gfortran python3-setuptools python3-scipy 

RUN curl https://bootstrap.pypa.io/get-pip.py --output get-pip.py
RUN python3.7 ./get-pip.py

RUN python3.7 -m pip install bottle==0.12.13 cherrypy==8.9.1 wsgi-request-logger prometheus_client

RUN python3.7 -m pip install --upgrade pip

RUN wget -O h5py-3.1.0-cp37-cp37m-linux_aarch64.whl "https://www.googleapis.com/drive/v3/files/1YVnEbNAOc85IVpirgEG7fGyU13b9hn8n?alt=media&key=AIzaSyA799IZVeyHS9OlAZFY4ktIKUA-bcWZNlo"
RUN python3.7 -m pip install h5py-3.1.0-cp37-cp37m-linux_aarch64.whl
RUN rm -rf h5py-3.1.0-cp37-cp37m-linux_aarch64.whl

RUN wget -O grpcio-1.34.1-cp37-cp37m-linux_aarch64.whl "https://www.googleapis.com/drive/v3/files/13oHrrLqvOmsRQPJSuaFtFQ2xsPqFTKVl?alt=media&key=AIzaSyA799IZVeyHS9OlAZFY4ktIKUA-bcWZNlo"
RUN python3.7 -m pip install grpcio-1.34.1-cp37-cp37m-linux_aarch64.whl
RUN rm -rf grpcio-1.34.1-cp37-cp37m-linux_aarch64.whl

RUN wget -O tensorflow-2.5.0-cp37-cp37m-linux_aarch64.whl "https://www.googleapis.com/drive/v3/files/11G81dJKC9-cRpjT7GaLpPX7D2-FbE6Ws?alt=media&key=AIzaSyA799IZVeyHS9OlAZFY4ktIKUA-bcWZNlo"
RUN python3.7 -m pip install tensorflow-*.whl
RUN rm -rf tensorflow-*.whl

RUN python3.7 -m pip install keras

ARG TARGETPLATFORM
ARG BUILDPLATFORM

# Allows you to add additional packages via build-arg
ARG ADDITIONAL_PACKAGE

COPY --from=watchdog /fwatchdog /usr/bin/fwatchdog
RUN chmod +x /usr/bin/fwatchdog
RUN apt-get update \
    && apt-get install -y ca-certificates ${ADDITIONAL_PACKAGE} \
    && rm -rf /var/lib/apt/lists/

# Add non root user
RUN groupadd app && useradd -r -g app app

WORKDIR /home/app/

COPY index.py           .
COPY requirements.txt   .

RUN chown -R app /home/app && \
    mkdir -p /home/app/python && chown -R app /home/app
#USER app
#USER 1000
ENV PATH=$PATH:/home/app/.local/bin:/home/app/python/bin/
ENV PYTHONPATH=$PYTHONPATH:/home/app/python

RUN python3.7 -m pip install -r requirements.txt --target=/home/app/python

RUN mkdir -p function
RUN touch ./function/__init__.py

WORKDIR /home/app/function/
COPY function/requirements.txt	.

COPY  data /tmp/data
COPY  har /tmp/har

RUN python3.7 -m pip install -r requirements.txt --target=/home/app/python

WORKDIR /home/app/

#USER root
#USER 1000

COPY function           function

# Allow any user-id for OpenShift users.
RUN chown -R app:app ./ && \
    chmod -R 777 /home/app/python

#USER app
#USER 1000

ENV fprocess="python3.7 index.py"
EXPOSE 8080

HEALTHCHECK --interval=3s CMD [ -e /tmp/.lock ] || exit 1

CMD ["fwatchdog"]
