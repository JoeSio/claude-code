FROM mcr.microsoft.com/devcontainers/universal:linux

ARG TZ
ENV TZ="$TZ"
ARG USERNAME=codespace

# 1. System-level tools (Root)
USER root
RUN apt-get update && apt-get install -y --no-install-recommends \
    bubblewrap fzf iproute2 dnsutils libnspr4 ripgrep \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# 2. Setup Directories & Permissions (Grouped)
RUN mkdir -p /usr/local/share/npm-global /commandhistory /workspace /myapp \
    /home/$USERNAME/.claude /home/$USERNAME/.codex /home/$USERNAME/.qwen /home/$USERNAME/.kimi \
    && touch /commandhistory/.bash_history \
    && chown -R $USERNAME:$USERNAME /usr/local/share/npm-global /commandhistory /workspace /myapp \
    && chown -R $USERNAME:$USERNAME /home/$USERNAME/.claude /home/$USERNAME/.codex /home/$USERNAME/.qwen /home/$USERNAME/.kimi

# 3. Global Configs
ENV DEVCONTAINER=true \
    EDITOR=nano \
    VISUAL=nano

# Set PATH to include our custom global bin AND the local bin
ENV PATH="/usr/local/share/npm-global/bin:/home/$USERNAME/.local/bin:$PATH"

# Force environment variables into shell profiles for interactive sessions
RUN echo "export NPM_CONFIG_PREFIX=/usr/local/share/npm-global" >> /home/$USERNAME/.zshrc \
    && echo "export NPM_CONFIG_PREFIX=/usr/local/share/npm-global" >> /home/$USERNAME/.bashrc \
    && echo "export PATH=\"/usr/local/share/npm-global/bin:/home/$USERNAME/.local/bin:\$PATH\"" >> /home/$USERNAME/.zshrc \
    && echo "export PATH=\"/usr/local/share/npm-global/bin:/home/$USERNAME/.local/bin:\$PATH\"" >> /home/$USERNAME/.bashrc

# 4. Versioned binary installs (Git Delta)
ARG GIT_DELTA_VERSION=0.18.2
RUN ARCH=$(dpkg --print-architecture) && \
    wget "https://github.com/dandavison/delta/releases/download/${GIT_DELTA_VERSION}/git-delta_${GIT_DELTA_VERSION}_${ARCH}.deb" && \
    dpkg -i "git-delta_${GIT_DELTA_VERSION}_${ARCH}.deb" && \
    rm "git-delta_${GIT_DELTA_VERSION}_${ARCH}.deb"

# 5. User-level tools
USER $USERNAME
WORKDIR /workspace

ARG CLAUDE_CODE_VERSION=latest
ARG CODEX_VERSION=latest
ARG QWEN_CODE_VERSION=latest
ARG KIMI_CLI_VERSION=latest

# Install Node 22 and the AI tools
# We 'unset' the prefix temporarily so NVM allows us to source it
RUN bash -c "unset NPM_CONFIG_PREFIX && . /usr/local/share/nvm/nvm.sh && \
    nvm install 22 && \
    nvm alias default 22 && \
    nvm use 22 && \
    npm install -g --prefix /usr/local/share/npm-global \
    @anthropic-ai/claude-code@${CLAUDE_CODE_VERSION} \
    @openai/codex@${CODEX_VERSION} \
    @qwen-code/qwen-code@${QWEN_CODE_VERSION}"

# UV and Kimi CLI
USER root
RUN curl -LsSf https://astral.sh/uv/install.sh | env UV_INSTALL_DIR=/usr/local/bin sh

USER $USERNAME
RUN uv tool install kimi-cli@${KIMI_CLI_VERSION}

# 6. Final Script Setup
USER root
COPY --chown=root:root scripts/* /usr/local/bin/
RUN chmod +x /usr/local/bin/*

USER $USERNAME