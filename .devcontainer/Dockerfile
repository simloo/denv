FROM mcr.microsoft.com/devcontainers/go:1-1.21-bullseye as base

RUN apt-get update && apt-get install -y \
    dnsutils && \
    rm -rf /var/lib/apt/lists/* && \
    apt-get clean

FROM base as build
WORKDIR /build

FROM build as pre-commit
ARG PRE_COMMIT_VERSION=3.4.0
RUN curl -L https://github.com/pre-commit/pre-commit/releases/download/v${PRE_COMMIT_VERSION}/pre-commit-${PRE_COMMIT_VERSION}.pyz -o pre-commit && \
    chmod +x pre-commit

FROM build as helm
ARG TARGETARCH
ARG HELM_VERSION=v3.12.3

RUN curl -sL https://get.helm.sh/helm-${HELM_VERSION}-linux-${TARGETARCH}.tar.gz | tar -xz --strip-components=1 linux-${TARGETARCH}/helm && \
    chmod +x helm

FROM build as kind
ARG TARGETOS
ARG TARGETARCH
ARG KIND_VERSION=v0.20.0
RUN curl -sL https://kind.sigs.k8s.io/dl/${KIND_VERSION}/kind-${TARGETOS}-${TARGETARCH} -o kind && \
    chmod +x kind

FROM build as act
ARG TARGETARCH
ARG ACT_VERSION=v0.2.53
RUN case $TARGETARCH in \
    amd64) ARCH="x86_64" ;; \
    arm64) ARCH="arm64" ;; \
    *) echo "Unsupported architecture: ${TARGETARCH}" && exit 1 ;; \
    esac && \
    curl -sL https://github.com/nektos/act/releases/download/${ACT_VERSION}/act_Linux_${ARCH}.tar.gz | tar -xz act && \
    chmod +x act

FROM build as k9s
ARG TARGETARCH
ARG K9S_VERSION=v0.28.2
RUN curl -sL https://github.com/derailed/k9s/releases/download/${K9S_VERSION}/k9s_Linux_${TARGETARCH}.tar.gz | tar -xz k9s && \
    chmod +x k9s

FROM build as kubectl
ARG TARGETOS
ARG TARGETARCH
ARG KUBECTL_VERSION=v1.28.1
RUN curl -LO https://dl.k8s.io/release/${KUBECTL_VERSION}/bin/${TARGETOS}/${TARGETARCH}/kubectl && \
    chmod +x kubectl

FROM build as crossplane
ARG TARGETOS
ARG TARGETARCH
ARG CROSSPLANE_VERSION=v1.13.2
RUN curl -L https://releases.crossplane.io/stable/${CROSSPLANE_VERSION}/bin/${TARGETOS}_${TARGETARCH}/crank -o crossplane && \
    chmod +x crossplane

FROM build as kuttl
ARG TARGETOS
ARG TARGETARCH
ARG KUTTL_VERSION=0.15.0
RUN case "${TARGETARCH}" in \
    amd64) ARCH="x86_64" ;; \
    arm64) ARCH="arm64" ;; \
    *) echo "Unsupported architecture: ${TARGETARCH}" && exit 1 ;; \
    esac && \
    curl -L https://github.com/kudobuilder/kuttl/releases/download/v${KUTTL_VERSION}/kubectl-kuttl_${KUTTL_VERSION}_${TARGETOS}_${ARCH} -o kuttl && \
    chmod +x kuttl

FROM build as up
ARG TARGETOS
ARG TARGETARCH
ARG UP_VERSION=v0.21.0
RUN curl -LO https://cli.upbound.io/stable/${UP_VERSION}/bin/${TARGETOS}_${TARGETARCH}/up && \
    chmod +x up

FROM build as terraform
ARG TARGETOS
ARG TARGETARCH
ARG TERRAFORM_VERSION=1.5.7
RUN curl -sLO https://releases.hashicorp.com/terraform/${TERRAFORM_VERSION}/terraform_${TERRAFORM_VERSION}_linux_${TARGETARCH}.zip && \
    unzip terraform_${TERRAFORM_VERSION}_linux_${TARGETARCH}.zip

FROM build as gh
ARG TARGETOS
ARG TARGETARCH
ARG GH_VERSION=2.35.0
RUN curl -sL https://github.com/cli/cli/releases/download/v${GH_VERSION}/gh_${GH_VERSION}_linux_${TARGETARCH}.tar.gz | tar -xz --strip-components=2 gh_${GH_VERSION}_linux_${TARGETARCH}/bin/gh

FROM build as yq
ARG TARGETOS
ARG TARGETARCH
ARG YQ_VERSION=4.35.2
RUN curl -sL https://github.com/mikefarah/yq/releases/download/v${YQ_VERSION}/yq_linux_${TARGETARCH} -o yq && \
    chmod +x yq

FROM build as awscliv2
ARG TARGETARCH
ARG AWSCLI_VERSION=2.13.33
RUN case $TARGETARCH in \
    amd64) ARCH=x86_64 ;; \
    arm64) ARCH=aarch64 ;; \
    *) echo "Unsupported architecture: $TARGETARCH" >&2; exit 1 ;; \
    esac && \
    curl -sL "https://awscli.amazonaws.com/awscli-exe-linux-${ARCH}-${AWSCLI_VERSION}.zip" -o "awscliv2.zip" && \
    unzip awscliv2.zip && \
    sudo ./aws/install

FROM base as packages
COPY --from=pre-commit /build/pre-commit /usr/local/bin/
COPY --from=helm /build/helm /usr/local/bin/
COPY --from=kind /build/kind /usr/local/bin/
COPY --from=act /build/act /usr/local/bin/
COPY --from=k9s /build/k9s /usr/local/bin/
COPY --from=kubectl /build/kubectl /usr/local/bin/
COPY --from=crossplane /build/crossplane /usr/local/bin/kubectl-crossplane
COPY --from=kuttl /build/kuttl /usr/local/bin/kubectl-kuttl
COPY --from=up /build/up /usr/local/bin/
COPY --from=terraform /build/terraform /usr/local/bin/
COPY --from=gh /build/gh /usr/local/bin/
COPY --from=yq /build/yq /usr/local/bin/
COPY --from=awscliv2 /usr/local/aws-cli/ /usr/local/aws-cli/



FROM packages as final
ENV PATH="/usr/local/aws-cli/v2/current/bin:${PATH}"
