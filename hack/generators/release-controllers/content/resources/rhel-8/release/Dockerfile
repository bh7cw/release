# Serves as a standard build environment for OpenShift builds. It is based on the
# ART golang builder and has packages layered on top of support CI only use cases
# (e.g. compiling test cases) that simply don't occur downstream.

FROM replaced-by-buildconfig
ARG MAJOR
ARG MINOR
ENV OPENSHIFT_CI=true

# Install common CI tools and epel for packages like tito.
RUN yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm && \
    INSTALL_PKGS="bc procps-ng util-linux bind-utils bsdtar createrepo_c device-mapper device-mapper-persistent-data e2fsprogs ethtool file findutils gcc git glib2-devel gpgme gpgme-devel hostname iptables jq krb5-devel libassuan libassuan-devel libseccomp-devel libvirt-devel lsof make mercurial nmap-ncat openssl protobuf-compiler protobuf-devel rsync socat systemd-devel tar tito tree wget which xfsprogs zip goversioninfo" && \
    yum install -y $INSTALL_PKGS && \
    rpm -V $INSTALL_PKGS && \
    yum clean all && \
    touch /os-build-image && \
    git config --system user.name origin-release-container && \
    git config --system user.email origin-release@redhat.com

# Install common go tools upstream devs are expecting in CI.
# Pure cargo culting from https://github.com/openshift/release/blob/51d92eb6a6d730e932a5daf68829ca7936739904/projects/origin-release/golang-1.13/Dockerfile#L41
RUN go get golang.org/x/tools/cmd/cover \
        github.com/Masterminds/glide \
        golang.org/x/tools/cmd/goimports \
        github.com/tools/godep \
        golang.org/x/lint/golint \
        gotest.tools/gotestsum \
        github.com/openshift/release/tools/gotest2junit \
        github.com/openshift/imagebuilder/cmd/imagebuilder && \
    GO111MODULE=on go get gotest.tools/gotestsum@v0.5.2 && \
    mv $GOPATH/bin/* /usr/bin/ && \
    rm -rf $GOPATH/* $GOPATH/.cache && \
    mkdir $GOPATH/bin && \
    mkdir -p /go/src/github.com/openshift/origin && \
    ln -s /usr/bin/imagebuilder $GOPATH/bin/imagebuilder && \
    ln -s /usr/bin/goimports $GOPATH/bin/goimports && \
    curl -L https://github.com/golang/dep/releases/download/v0.5.4/dep-linux-amd64 > /usr/bin/dep && \
    chmod +x /usr/bin/dep

# make go related directories writeable since builds in CI will run as non-root.
RUN mkdir -p $GOPATH && \
    chmod g+xw -R $GOPATH && \
    chmod g+xw -R $(go env GOROOT)

WORKDIR /go/src/github.com/openshift/origin
LABEL io.k8s.display-name="OpenShift ${MAJOR}.${MINOR} RHEL-8 Golang Release Image" \
      io.k8s.description="This is the standard Release environment for building OpenShift ${MAJOR}.${MINOR} RHEL-8 images."
