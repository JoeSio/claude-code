FROM mcr.microsoft.com/devcontainers/universal:linux

ARG TZ
ENV TZ="$TZ"

ARG USERNAME=codespace
USER root

# Install project-specific tools not included in the universal image
RUN apt-get update && apt-get install -y --no-install-recommends \
  bubblewrap \
  fzf \
  iproute2 \
  dnsutils \
  libnspr4 \
  && apt-get clean && rm -rf /var/lib/apt/lists/*

# Ensure global npm install directory exists
RUN mkdir -p /usr/local/share/npm-global && \
  chown -R $USERNAME:$USERNAME /usr/local/share/npm-global

# Persist bash history
RUN SNIPPET="export PROMPT_COMMAND='history -a' && export HISTFILE=/commandhistory/.bash_history" \
  && mkdir /commandhistory \
  && touch /commandhistory/.bash_history \
  && chown -R $USERNAME /commandhistory

ENV DEVCONTAINER=true

# Create workspace and config directories
RUN mkdir -p /workspace /home/$USERNAME/.claude /home/$USERNAME/.codex /home/$USERNAME/.qwen /home/$USERNAME/.kimi /myapp && \
  chown -R $USERNAME:$USERNAME /workspace /home/$USERNAME/.claude /home/$USERNAME/.codex /home/$USERNAME/.qwen /home/$USERNAME/.kimi /myapp

WORKDIR /workspace

ARG CLAUDE_CODE_VERSION=latest
ARG CODEX_VERSION=latest
ARG QWEN_CODE_VERSION=latest
ARG KIMI_CLI_VERSION=latest
ARG GIT_DELTA_VERSION=0.18.2

RUN ARCH=$(dpkg --print-architecture) && \
  wget "https://github.com/dandavison/delta/releases/download/${GIT_DELTA_VERSION}/git-delta_${GIT_DELTA_VERSION}_${ARCH}.deb" && \
  dpkg -i "git-delta_${GIT_DELTA_VERSION}_${ARCH}.deb" && \
  rm "git-delta_${GIT_DELTA_VERSION}_${ARCH}.deb"

USER $USERNAME

# Add custom global npm path to PATH
ENV PATH=$PATH:/usr/local/share/npm-global/bin:/home/$USERNAME/.local/bin

# Default editor
ENV EDITOR=nano
ENV VISUAL=nano

# Install global npm packages
RUN NPM_CONFIG_PREFIX=/usr/local/share/npm-global npm install -g @anthropic-ai/claude-code@${CLAUDE_CODE_VERSION}
RUN NPM_CONFIG_PREFIX=/usr/local/share/npm-global npm install -g @openai/codex@${CODEX_VERSION}
RUN NPM_CONFIG_PREFIX=/usr/local/share/npm-global npm install -g @qwen-code/qwen-code@${QWEN_CODE_VERSION}
RUN NPM_CONFIG_PREFIX=/usr/local/share/npm-global npm install -g @playwright/cli@latest
RUN playwright install

# Install uv (as root so it can write to /usr/local/bin)
USER root
RUN curl -LsSf https://astral.sh/uv/install.sh | env UV_INSTALL_DIR=/usr/local/bin sh
USER $USERNAME

# Install Kimi CLI
RUN uv tool install kimi-cli@${KIMI_CLI_VERSION}

# Copy custom global scripts
COPY --chown=root:root scripts/* /usr/local/bin/

# Ensure custom scripts are executable
USER root
RUN chmod +x /usr/local/bin/*
USER $USERNAME
