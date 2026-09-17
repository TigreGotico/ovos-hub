FROM ghcr.io/oscillatelabsllc/ovos-rust-messagebus:v1.1.2

HEALTHCHECK --interval=60s --timeout=10s --retries=3 --start-period=60s \
    CMD curl --header "Connection: Upgrade" --header "Upgrade: Websocket" http://127.0.0.1:8181/core || kill 1
