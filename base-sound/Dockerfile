FROM jarbasai/ovos-base:latest

COPY files/client.conf /etc/pulse/client.conf

USER root

SHELL ["/bin/bash", "-c"]

RUN apt-get update \
  && apt-get install -o Dpkg::Options::="--force-confold" -y --no-install-recommends --no-install-suggests \
  gpg alsa-utils libasound2-plugins flac libportaudio2 portaudio19-dev \
  pulseaudio-utils pipewire pipewire-alsa mpg123 music123 \
  sox swig libsox-fmt-all procps libasound2-dev \
  mpv libespeak-ng1 \
  build-essential python3-dev \
  && uv pip install pyalsaaudio \
  && mkdir -p "/home/${USER}/.config/mycroft/apps" "/home/${USER}/.config/pulse" \
  && chown "${USER}:${USER}" -R "/home/${USER}" \
  && usermod -a -G audio "$USER"

# install espeak from source to ensure latest lang support
# used by various TTS/STT plugins
RUN apt-get update \
    && apt-get install -y cmake \
    && git clone https://github.com/espeak-ng/espeak-ng.git /tmp/espeak-ng \
    && cd /tmp/espeak-ng \
    && cmake -B build -DCMAKE_BUILD_TYPE=Release \
    && cmake --build build -j"$(nproc)" \
    && cmake --install build \
    && ldconfig \
    && apt-get --purge autoremove -y \
    && apt-get clean \
  && rm -rf /var/lib/apt/lists/* /var/cache/apt/archives/* /var/log/{apt,dpkg.log} /tmp/* /var/tmp/*


USER "$USER"
