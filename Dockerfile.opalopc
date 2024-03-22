FROM ubuntu:jammy

# Install dependencies
RUN apt-get update && apt-get install -y \
    curl \
    libicu70

# Install opalopc http://opalopc.com/docs/get-started/install
RUN curl -LO "https://dl.opalopc.com/release/$(curl -L -s https://dl.opalopc.com/release/stable.txt)/bin/linux/amd64/opalopc" \
    && install -o root -g root -m 0755 opalopc /usr/local/bin/opalopc
