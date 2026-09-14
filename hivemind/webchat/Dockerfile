FROM jarbasai/ovos-base:latest
COPY --chown=${USER}:${USER} files/requirements.txt /tmp/requirements.txt

USER root

RUN uv pip install -r /tmp/requirements.txt && rm -rf /tmp/requirements.txt

USER "$USER"

# hivemind-webchat binds 127.0.0.1 by default; the container must listen on
# every interface so the published port 9090 answers.
ENV WEBCHAT_HOST=0.0.0.0

ENTRYPOINT hivemind-webchat

WORKDIR /home/${USER}

EXPOSE 9090
