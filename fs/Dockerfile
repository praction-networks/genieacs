# ----------- Build Stage -----------
FROM node:24-bookworm AS builder

WORKDIR /opt
ARG GENIEACS_VERSION=v1.2.13

# Install build dependencies
RUN apt-get update && \
    apt-get install -y git python3 make g++ && \
    rm -rf /var/lib/apt/lists/*

# Clone and build GenieACS
RUN git clone --depth 1 --branch "${GENIEACS_VERSION}" https://github.com/genieacs/genieacs.git
WORKDIR /opt/genieacs
RUN npm ci --unsafe-perm && npm run build

# ----------- Runtime Stage -----------
FROM debian:bookworm-slim

# Security patch and minimal install
RUN apt-get update && apt-get upgrade -y && \
    apt-get install -y --no-install-recommends ca-certificates && \
    rm -rf /var/lib/apt/lists/*

# Copy built files from builder
COPY --from=builder /usr/local /usr/local
COPY --from=builder /opt/genieacs /opt/genieacs

# Create user and set permissions
RUN useradd --system --home /opt/genieacs genieacs && \
    mkdir -p /opt/genieacs/ext /var/log/genieacs && \
    chown -R genieacs:genieacs /opt/genieacs /var/log/genieacs

USER genieacs
WORKDIR /opt/genieacs

EXPOSE 7547

CMD ["node", "dist/bin/genieacs-fs"]

