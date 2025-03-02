FROM docker.io/library/golang:1.18.8-alpine3.16 as build
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY cmd ./cmd
COPY pkg ./pkg
# wsl-gvproxy.exe is compiled as a windows GUI to support backgrounding
RUN GOOS=windows go build -ldflags '-H=windowsgui' -o bin/wsl-gvproxy.exe ./cmd/gvproxy && \
    GOOS=linux CGO_ENABLED=0 go build -ldflags '-s -w' -o bin/wsl-vm ./cmd/vm && \
    find ./bin -type f -exec sha256sum {} \;

FROM docker.io/library/golang:1.18.8-alpine3.16 as licenses
RUN apk add --no-cache git && \
    apk list --installed && \
    go install github.com/google/go-licenses@v1.0.0
WORKDIR /app
COPY go.mod go.sum LICENSE ./
COPY cmd ./cmd
COPY pkg ./pkg
RUN go-licenses save ./cmd/gvproxy --save_path ./licenses/gvproxy && \
    go-licenses save ./cmd/vm --save_path ./licenses/vm

FROM docker.io/library/alpine:3.16.3
RUN apk add --no-cache openrc iptables && \
    apk list --installed
ARG REF=https://example.com/
ARG VERSION=v0.0.0
WORKDIR /app
COPY --from=build /app/bin /files/
COPY --from=licenses /app/licenses/gvproxy /app/licenses/vm /licenses/
COPY ./distro/conf/udhcpc.conf /etc/udhcpc/udhcpc.conf
COPY ./distro/scripts/ ./wsl-vpnkit ./LICENSE ./
RUN echo "$REF" > /app/ref && \
    echo "$VERSION" > /app/version && \
    find /files -type f -exec sha256sum {} \; && \
    ln -s /files/wsl-vm /app/wsl-vpnkit /usr/sbin/ && \
    ln -s /app/wsl-vpnkit.service /etc/init.d/wsl-vpnkit && \
    ln -s /app/startup.sh /etc/profile.d/
