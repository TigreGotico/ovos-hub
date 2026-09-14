FROM docker.io/rocm/pytorch:latest

ARG CHANNEL=alpha
ARG USER=ovos
ARG USER_UID=1000
ARG USER_GID=1000

ENV USER="$USER"
ENV OVOS_CHANNEL="$CHANNEL"
ENV IS_OVOS_CONTAINER=true
ENV DEBIAN_FRONTEND=noninteractive
ENV PYTHONUNBUFFERED=1
ENV PYTHONDONTWRITEBYTECODE=1
ENV PIP_NO_CACHE_DIR=1
ENV PIP_DISABLE_PIP_VERSION_CHECK=1
# Every image installs into this virtual environment. Debian marks its own
# interpreter externally managed (PEP 668), so an install into it is refused,
# and --break-system-packages only moves the damage into the system Python.
ENV VIRTUAL_ENV=/opt/ovos
ENV PATH="/opt/ovos/bin:$PATH"
# use OVOS alpha channel
ENV PIP_PRE=1
ENV UV_PRERELEASE=allow
ENV UV_CONSTRAINT=https://raw.githubusercontent.com/OpenVoiceOS/ovos-releases/refs/heads/main/constraints-${OVOS_CHANNEL}.txt

SHELL ["/bin/bash", "-c"]

# Install packages first (changes more frequently than user creation)
RUN apt-get update \
  && apt-get install -y --no-install-recommends --no-install-suggests \
  ca-certificates \
  git \
  curl \
  python3-pip \
  python3-venv \
  && c_rehash \
  && apt-get --purge autoremove -y \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/* /var/cache/apt/archives/* /var/log/{apt,dpkg.log} /tmp/* /var/tmp/*

# Create user with specific UID/GID for better security and consistency
RUN groupmod -n "$USER" ubuntu \
 && usermod -l "$USER" -d "/home/$USER" -m ubuntu \
 && usermod -g "$USER" "$USER" \
 && chown -R "$USER:$USER" "/home/$USER"

# Setup Python environment: uv as a standalone binary, never through the
# Debian interpreter, then the virtual environment every child image uses.
ADD https://astral.sh/uv/0.12.13/install.sh /tmp/uv-install.sh
RUN env UV_INSTALL_DIR=/usr/local/bin INSTALLER_NO_MODIFY_PATH=1 sh /tmp/uv-install.sh \
  && rm -f /tmp/uv-install.sh \
  && uv --version \
  && uv venv --python python3 "$VIRTUAL_ENV" \
  && uv pip install --upgrade setuptools wheel \
  && chown -R "$USER_UID:$USER_GID" "$VIRTUAL_ENV"

# health check script
ENV OVOS_NAMESPACE=mycroft
COPY files/ovos-hc.py "/home/${USER}/ovos-hc"
RUN chmod +x "/home/${USER}/ovos-hc" && chown "${USER}:${USER}" "/home/${USER}/ovos-hc"
HEALTHCHECK --interval=60s --timeout=10s --retries=3 --start-period=60s \
  CMD python3 "/home/${USER}/ovos-hc" || exit 1

# ensure permissions of home dir
RUN chown "${USER}:${USER}" -R "/home/${USER}"

USER "$USER"

WORKDIR "/home/${USER}"
