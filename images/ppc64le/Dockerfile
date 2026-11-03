# ppc64le images built from binaries produced by hack/build-go.sh (no Bazel).
# Build context is the repo root; see Dockerfile.dockerignore.
ARG VERSION=v1.10.0-beta.0-260-g1165f2db5e

FROM quay.io/centos/centos:stream10 AS base

FROM base AS virt-operator
ARG VERSION
COPY _out/cmd/virt-operator/virt-operator-${VERSION}-linux-ppc64le /usr/bin/virt-operator
COPY _out/cmd/virt-operator/.version /.version
USER 1001
ENTRYPOINT ["/usr/bin/virt-operator"]

FROM base AS virt-api
ARG VERSION
COPY _out/cmd/virt-api/virt-api-${VERSION}-linux-ppc64le /usr/bin/virt-api
COPY _out/cmd/virt-api/.version /.version
USER 1001
ENTRYPOINT ["/usr/bin/virt-api"]

FROM base AS virt-controller
ARG VERSION
COPY _out/cmd/virt-controller/virt-controller-${VERSION}-linux-ppc64le /usr/bin/virt-controller
COPY _out/cmd/virt-controller/.version /.version
USER 1001
ENTRYPOINT ["/usr/bin/virt-controller"]

FROM base AS virt-handler
ARG VERSION
RUN dnf -y --enablerepo=crb install --setopt=install_weak_deps=False \
      passt qemu-img findutils iproute nftables procps-ng selinux-policy \
      selinux-policy-targeted tar util-linux xorriso && dnf clean all && test -x /usr/bin/passt-repair && chmod 0744 /usr/bin/passt-repair
RUN (getent group qemu || groupadd -g 107 qemu) && \
    (getent passwd qemu || useradd -u 107 -g qemu -s /bin/bash qemu) && \
    test "$(id -u qemu)" = 107
COPY cmd/virt-handler/nsswitch.conf /etc/nsswitch.conf
COPY _out/cmd/virt-handler/virt-handler-${VERSION}-linux-ppc64le /usr/bin/virt-handler
COPY _out/cmd/virt-chroot/virt-chroot-${VERSION}-linux-ppc64le /usr/bin/virt-chroot
COPY _out/cmd/virt-handler/.version /.version
ENTRYPOINT ["/usr/bin/virt-handler"]

FROM base AS virt-launcher
ARG VERSION
RUN printf '[kvm-power]\nname=kvm-power\nbaseurl=https://mirror.stream.centos.org/SIGs/10-stream/virt/ppc64le/kvm-power/\ngpgcheck=0\nenabled=1\n' > /etc/yum.repos.d/kvm-power.repo
RUN dnf -y --enablerepo=crb install --setopt=install_weak_deps=False \
      libvirt-client-12.5.0-1.virt.el10 libvirt-daemon-driver-qemu-12.5.0-1.virt.el10 \
      libvirt-libs-12.5.0-1.virt.el10 libnbd passt qemu-kvm-core qemu-kvm-device-usb-host \
      swtpm-tools edk2-ovmf qemu-kvm-device-display-virtio-gpu \
      qemu-kvm-device-display-virtio-gpu-pci nftables nmap-ncat procps-ng tar virtiofsd \
      xorriso libcap && dnf clean all
RUN command -v qemu-img virtqemud virsh swtpm virtiofsd setcap && test -x /usr/libexec/qemu-kvm
RUN (getent group qemu || groupadd -g 107 qemu) && \
    (getent passwd qemu || useradd -u 107 -g qemu -d /home/qemu -m -s /bin/bash qemu) && \
    test "$(id -u qemu)" = 107
COPY cmd/virt-launcher/nsswitch.conf /etc/nsswitch.conf
COPY cmd/virt-launcher/qemu.conf cmd/virt-launcher/virtqemud.conf /etc/libvirt/
COPY cmd/virt-launcher/node-labeller/node-labeller.sh /usr/bin/node-labeller.sh
COPY _out/cmd/virt-launcher/virt-launcher-${VERSION}-linux-ppc64le /usr/bin/virt-launcher
COPY _out/cmd/virt-launcher-monitor/virt-launcher-monitor-${VERSION}-linux-ppc64le /usr/bin/virt-launcher-monitor
COPY _out/cmd/container-disk-v2alpha/container-disk /usr/bin/container-disk
COPY _out/cmd/virt-freezer/virt-freezer-${VERSION}-linux-ppc64le /usr/bin/virt-freezer
COPY _out/cmd/virt-probe/virt-probe-${VERSION}-linux-ppc64le /usr/bin/virt-probe
COPY _out/cmd/virt-tail/virt-tail-${VERSION}-linux-ppc64le /usr/bin/virt-tail
COPY _out/cmd/libvirt-hook-client/libvirt-hook-client-${VERSION}-linux-ppc64le /usr/bin/libvirt-hook-client
COPY _out/cmd/libvirt-hook-client/libvirt-hook-client-${VERSION}-linux-ppc64le /etc/libvirt/hooks/qemu
COPY _out/cmd/virt-launcher/.version /.version
RUN chmod 0755 /etc/libvirt /usr/bin/node-labeller.sh /etc/libvirt/hooks/qemu && \
    chown 107:107 /etc/libvirt/hooks/qemu && \
    mkdir -p /usr/lib64/qemu-kvm && \
    setcap cap_net_bind_service+ep /usr/bin/virt-launcher-monitor
USER 107
ENTRYPOINT ["/usr/bin/virt-launcher"]

FROM base AS virt-exportproxy
ARG VERSION
COPY _out/cmd/virt-exportproxy/virt-exportproxy-${VERSION}-linux-ppc64le /usr/bin/virt-exportproxy
COPY _out/cmd/virt-exportproxy/.version /.version
USER 1001
ENTRYPOINT ["/usr/bin/virt-exportproxy"]

FROM base AS virt-exportserver
ARG VERSION
RUN dnf -y --enablerepo=crb install --setopt=install_weak_deps=False \
      tar libnbd nbdkit nbdkit-basic-filters nbdkit-basic-plugins nbdkit-curl-plugin nbdkit-server qemu-img \
    && dnf clean all
RUN groupadd -g 107 nonroot-user && useradd -u 107 -g 107 -d /home/nonroot-user -m -s /bin/bash nonroot-user
COPY _out/cmd/virt-exportserver/virt-exportserver-${VERSION}-linux-ppc64le /usr/bin/virt-exportserver
COPY _out/cmd/virt-exportserver/.version /.version
USER 1001
ENTRYPOINT ["/usr/bin/virt-exportserver"]

FROM base AS virt-synchronization-controller
ARG VERSION
COPY _out/cmd/synchronization-controller/synchronization-controller-${VERSION}-linux-ppc64le /usr/bin/virt-synchronization-controller
COPY _out/cmd/synchronization-controller/.version /.version
USER 1001
ENTRYPOINT ["/usr/bin/virt-synchronization-controller"]
