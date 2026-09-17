FROM ghcr.io/oscillatelabsllc/ovos-skill-config-tool:latest

# The parent image has a venv at /opt/venv but no uv and no git.
USER root
ADD https://astral.sh/uv/0.12.13/install.sh /tmp/uv-install.sh
RUN env UV_INSTALL_DIR=/usr/local/bin INSTALLER_NO_MODIFY_PATH=1 sh /tmp/uv-install.sh \
  && rm -f /tmp/uv-install.sh \
  && uv pip install --python /opt/venv/bin/python \
    https://github.com/JarbasAl/ovos-skill-config-tool/archive/refs/heads/main.tar.gz \
  && chown -R appuser:appuser /opt/venv
USER appuser

HEALTHCHECK --interval=60s --timeout=10s --retries=3 --start-period=60s \
    CMD curl --header "Connection: Upgrade" --header "Upgrade: Websocket" http://127.0.0.1:8000/core || kill 1
