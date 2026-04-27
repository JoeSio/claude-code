FROM node:20

ARG TZ
ENV TZ="$TZ"
ENV LANG=C.UTF-8
ENV LC_ALL=C.UTF-8

ARG CLAUDE_CODE_VERSION=latest
ARG CODEX_VERSION=latest
ARG QWEN_CODE_VERSION=latest
ARG KIMI_CLI_VERSION=latest

# Install basic development tools and iptables/ipset
RUN apt-get update && apt-get install -y --no-install-recommends \
  less \
  bubblewrap \
  git \
  procps \
  sudo \
  fzf \
  zsh \
  man-db \
  unzip \
  gnupg2 \
  gh \
  iptables \
  ipset \
  iproute2 \
  dnsutils \
  aggregate \
  jq \
  nano \
  vim \
  && apt-get clean && rm -rf /var/lib/apt/lists/*

# Ensure default node user has access to /usr/local/share
RUN mkdir -p /usr/local/share/npm-global && \
  chown -R node:node /usr/local/share

ARG USERNAME=node

# Persist bash history.
RUN SNIPPET="export PROMPT_COMMAND='history -a' && export HISTFILE=/commandhistory/.bash_history" \
  && mkdir /commandhistory \
  && touch /commandhistory/.bash_history \
  && chown -R $USERNAME /commandhistory

# Set `DEVCONTAINER` environment variable to help with orientation
ENV DEVCONTAINER=true

# Create workspace and config directories and set permissions
RUN mkdir -p /workspace /home/node/.claude /home/node/.codex /home/node/.qwen /home/node/.kimi /myapp && \
  chown -R node:node /workspace /home/node/.claude /home/node/.codex /home/node/.qwen /home/node/.kimi /myapp

WORKDIR /workspace

ARG NVM_VERSION=0.40.2
ARG GIT_DELTA_VERSION=0.18.2
RUN ARCH=$(dpkg --print-architecture) && \
  wget "https://github.com/dandavison/delta/releases/download/${GIT_DELTA_VERSION}/git-delta_${GIT_DELTA_VERSION}_${ARCH}.deb" && \
  sudo dpkg -i "git-delta_${GIT_DELTA_VERSION}_${ARCH}.deb" && \
  rm "git-delta_${GIT_DELTA_VERSION}_${ARCH}.deb"

# Set up non-root user
USER node

# Install global packages
ENV NPM_CONFIG_PREFIX=/usr/local/share/npm-global
ENV PATH=$PATH:/usr/local/share/npm-global/bin:/home/node/.local/bin

# Set the default shell to zsh rather than sh
ENV SHELL=/bin/zsh

# Set the default editor and visual
ENV EDITOR=nano
ENV VISUAL=nano

# Default powerline10k theme
ARG ZSH_IN_DOCKER_VERSION=1.2.0
RUN sh -c "$(wget -O- https://github.com/deluan/zsh-in-docker/releases/download/v${ZSH_IN_DOCKER_VERSION}/zsh-in-docker.sh)" -- \
  -p git \
  -p fzf \
  -a "source /usr/share/doc/fzf/examples/key-bindings.zsh" \
  -a "source /usr/share/doc/fzf/examples/completion.zsh" \
  -a "export PROMPT_COMMAND='history -a' && export HISTFILE=/commandhistory/.bash_history" \
  -x

# Install nvm
ENV NVM_DIR=/home/node/.nvm
RUN wget -qO- "https://raw.githubusercontent.com/nvm-sh/nvm/v${NVM_VERSION}/install.sh" | PROFILE=/dev/null bash \
  && printf '\n# nvm\nexport NVM_DIR="$HOME/.nvm"\n[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"\n[ -s "$NVM_DIR/bash_completion" ] && . "$NVM_DIR/bash_completion"\n' >> /home/node/.zshrc

# Install Claude
RUN NPM_CONFIG_PREFIX=/usr/local/share/npm-global npm install -g @anthropic-ai/claude-code@${CLAUDE_CODE_VERSION}

# Install Codex
RUN NPM_CONFIG_PREFIX=/usr/local/share/npm-global npm install -g @openai/codex@${CODEX_VERSION}

# Install Qwen Code
RUN NPM_CONFIG_PREFIX=/usr/local/share/npm-global npm install -g @qwen-code/qwen-code@${QWEN_CODE_VERSION}

# Install uv (as root so it can write to /usr/local/bin)
USER root
RUN curl -LsSf https://astral.sh/uv/install.sh | env UV_INSTALL_DIR=/usr/local/bin sh
USER node

# Install Kimi CLI
RUN uv tool install kimi-cli@${KIMI_CLI_VERSION}

# Copy and set up firewall script
COPY init-firewall.sh /usr/local/bin/
USER root
RUN chmod +x /usr/local/bin/init-firewall.sh && \
  echo "node ALL=(root) NOPASSWD: /usr/local/bin/init-firewall.sh" > /etc/sudoers.d/node-firewall && \
  chmod 0440 /etc/sudoers.d/node-firewall && \
  echo "node ALL=(root) NOPASSWD: ALL" > /etc/sudoers.d/node && \
  chmod 0440 /etc/sudoers.d/node
USER node
