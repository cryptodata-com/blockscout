# Use the official Ubuntu 22.04 image as the base image
FROM ubuntu:20.04

# Set environment variables
ENV DEBIAN_FRONTEND=noninteractive
ENV PATH="${PATH}:/root/.asdf/bin:/root/.asdf/shims"

# Update the package list and install required dependencies
RUN apt-get update && \
    apt-get install -y curl git wget unzip \
    build-essential autoconf m4 libncurses5-dev libwxgtk3.0-gtk3-dev \
    libwxgtk-webview3.0-gtk3-dev libgl1-mesa-dev libglu1-mesa-dev \
    libpng-dev libssh-dev unixodbc-dev xsltproc fop libxml2-utils \
    libncurses-dev openjdk-11-jdk && \
    apt-get clean

# Add erlang repos
RUN wget https://packages.erlang-solutions.com/erlang-solutions_2.0_all.deb && \
    wget https://packages.erlang-solutions.com/ubuntu/erlang_solutions.asc && \
    # install repo
    dpkg -i erlang-solutions_2.0_all.deb && \
    apt-key add erlang_solutions.asc && \
    rm erlang-solutions_2.0_all.deb && \
    # remove key
    rm erlang_solutions.asc

# Install NodeJS 
RUN curl -s https://deb.nodesource.com/setup_16.x | bash && \
    apt install nodejs -y

# Install Rust
RUN curl https://sh.rustup.rs -sSf | sh -s -- -y && \
    apt -y install cargo

# Install Erlang & Elixir
RUN apt-get install esl-erlang elixir -y

# Clone BlockScout and checkout the desired version
RUN git clone --branch xiden-dev https://github.com/cryptodata-com/blockscout /blockscout
WORKDIR /blockscout

# Install dependencies and build BlockScout
RUN mix local.hex --force && \
    mix local.rebar --force && \
    mix do deps.get, local.rebar --force, deps.compile, compile

# Compile the static assets
RUN cd apps/block_scout_web/assets/ && \
    npm install && \
    npm run deploy && \
    cd - && \
    mix phx.digest

# Expose the default BlockScout port
EXPOSE 4000


# Clean up
RUN apt-get autoremove -y && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Run BlockScout
CMD ["mix", "phx.server"]