FROM dockage/tor-privoxy

RUN apk --no-cache --update add curl
HEALTHCHECK --interval=60s --timeout=15s --start-period=20s \
            CMD curl -x socks5h://localhost:9050 -s https://check.torproject.org/api/ip | grep -qm1 true
