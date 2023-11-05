# # Use the official Debian-hosted Python image
# FROM python:3.9-slim-bullseye

# ARG DEBIAN_PACKAGES="build-essential"

# # Prevent apt from showing prompts
# ENV DEBIAN_FRONTEND=noninteractive
# ENV CLOUDSDK_PYTHON=3.9
# ENV CLOUDSDK_PYTHON_SITEPACKAGES=1

# # Ensure we have an up to date baseline, install dependencies and
# # create a user so we don't run the app as root
# RUN set -ex; \
#     for i in $(seq 1 8); do mkdir -p "/usr/share/man/man${i}"; done && \
#     apt-get update && \
#     apt-get upgrade -y && \
#     apt-get install -y --no-install-recommends $DEBIAN_PACKAGES && \
#     apt-get install -y --no-install-recommends software-properties-common apt-transport-https ca-certificates gnupg2 gnupg-agent gnupg lsb-release curl openssh-client && \
#     curl -s https://download.docker.com/linux/debian/gpg | apt-key add - && \
#     add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/debian $(lsb_release -cs) stable" && \
#     apt-get update && \
#     apt-get install -y --no-install-recommends docker-ce && \
#     apt-get clean && \
#     rm -rf /var/lib/apt/lists/* && \
#     pip install --no-cache-dir --upgrade pip && \
#     pip install pipenv && \
#     pip install openshift docker apache-libcloud && \
#     useradd -ms /bin/bash app -d /home/app -u 1000 -p "$(openssl passwd -1 passw0rd)" && \
#     usermod -aG docker app && \
#     mkdir -p /app && \
#     chown app:app /app


# WORKDIR /app

# # Add the rest of the source code. This is done last so we don't invalidate all
# # layers when we change a line of code.
# ADD --chown=app:app . /app

# # Entry point
# ENTRYPOINT ["/bin/bash","./docker-entrypoint.sh"]


# Use the official Ubuntu 20.04 image as the base
FROM ubuntu:20.04

# Set the environment variable for non-interactive installations
ENV DEBIAN_FRONTEND=noninteractive

# Install required dependencies
RUN apt-get update && \
    apt-get install -y curl apt-transport-https ca-certificates gnupg lsb-release openssh-client

# Add the Google Cloud SDK repository
RUN echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] https://packages.cloud.google.com/apt cloud-sdk main" | \
    tee -a /etc/apt/sources.list.d/google-cloud-sdk.list && \
    curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | \
    gpg --dearmor -o /usr/share/keyrings/cloud.google.gpg

RUN install -m 0755 -d /etc/apt/keyrings
RUN curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg
RUN chmod a+r /etc/apt/keyrings/docker.gpg

RUN echo "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
    "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
    tee /etc/apt/sources.list.d/docker.list > /dev/null
RUN apt-get update
RUN apt-get install -y docker-ce

# Install the Google Cloud SDK
RUN apt-get update && \
    apt-get install -y google-cloud-sdk

RUN curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg && \
    echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | tee /etc/apt/sources.list.d/kubernetes.list  && \
    curl https://baltocdn.com/helm/signing.asc | apt-key add -&& \
    echo "deb https://baltocdn.com/helm/stable/debian/ all main" | tee /etc/apt/sources.list.d/helm-stable-debian.list && \
    apt-get update && \
    apt-get install -y --no-install-recommends kubectl helm python3 python3-pip && \
    pip install openshift ansible docker apache-libcloud

RUN useradd -ms /bin/bash app -d /home/app -u 1000 -p "$(openssl passwd -1 passw0rd)" && \
    usermod -aG docker app && \
    mkdir -p /app && \
    chown app:app /app


# Set the working directory
WORKDIR /app
ADD --chown=app:app . /app

# Install ansible dependencies
RUN set -ex; \
    ansible-galaxy collection install community.general community.kubernetes

# Start a new shell to use the installed SDK
ENTRYPOINT ["/bin/bash","./docker-entrypoint.sh"]