# =============================================================================
# koderzi/dev — Universal Dev Container
# GPU-accelerated development environment with AI coding tools,
# Google Antigravity, Chrome, and mise-based project isolation.
# =============================================================================

FROM debian:bookworm

LABEL maintainer="koderzi"
LABEL description="Universal GPU dev container with AI tools, Antigravity, and mise"

# =============================================================================
# Environment Configuration
# =============================================================================
ENV DEBIAN_FRONTEND=noninteractive \
  TZ=UTC \
  LANG=en_US.UTF-8 \
  LANGUAGE=en_US:en \
  LC_ALL=en_US.UTF-8 \
  DONT_PROMPT_WSL_INSTALL=1

# =============================================================================
# 1. System Dependencies
# =============================================================================
RUN apt-get update && apt-get install -y --no-install-recommends \
  # Core utilities
  ca-certificates \
  curl \
  wget \
  bash \
  git \
  gnupg \
  sudo \
  procps \
  net-tools \
  locales \
  # Build essentials (for native npm packages, Python C extensions, etc.)
  build-essential \
  pkg-config \
  python3-dev \
  libicu-dev \
  # Chrome dependencies
  libglib2.0-dev \
  libnss3 \
  libatk1.0-0 \
  libatk-bridge2.0-0 \
  # GUI and IPC dependencies
  dbus-x11 \
  libx11-xcb1 \
  libcups2 \
  libdrm2 \
  libgtk-3-0 \
  libgbm1 \
  libasound2 \
  && rm -rf /var/lib/apt/lists/*

# =============================================================================
# 2. Locale Configuration
# =============================================================================
RUN sed -i '/en_US.UTF-8/s/^# //g' /etc/locale.gen && locale-gen

# =============================================================================
# 3. Create 'dev' user with sudo access (UID 1000)
# =============================================================================
RUN groupadd --gid 1000 dev \
  && useradd --uid 1000 --gid 1000 -m dev -s /bin/bash \
  && echo "dev ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/dev \
  && chmod 0440 /etc/sudoers.d/dev \
  && mkdir -p /home/dev/Desktop \
  && chown dev:dev /home/dev/Desktop

# =============================================================================
# 4. Install Google Chrome
# =============================================================================
RUN curl -fsSL https://dl.google.com/linux/linux_signing_key.pub \
  | gpg --dearmor -o /usr/share/keyrings/google-chrome.gpg \
  && echo "deb [arch=amd64 signed-by=/usr/share/keyrings/google-chrome.gpg] \
  https://dl.google.com/linux/chrome/deb/ stable main" \
  > /etc/apt/sources.list.d/google-chrome.list \
  && apt-get update \
  && apt-get install -y google-chrome-stable \
  && rm -rf /var/lib/apt/lists/*

# =============================================================================
# 5. Install Google Antigravity (via official APT repo)
#    Reference: https://github.com/rawritude/antigravity-remote-docker
# =============================================================================
RUN mkdir -p /etc/apt/keyrings \
  && curl -fsSL https://us-central1-apt.pkg.dev/doc/repo-signing-key.gpg | \
  gpg --dearmor --yes -o /etc/apt/keyrings/antigravity-repo-key.gpg \
  && echo "deb [signed-by=/etc/apt/keyrings/antigravity-repo-key.gpg] \
  https://us-central1-apt.pkg.dev/projects/antigravity-auto-updater-dev/ \
  antigravity-debian main" | \
  tee /etc/apt/sources.list.d/antigravity.list > /dev/null \
  && apt-get update \
  && apt-get install -y antigravity \
  && rm -rf /var/lib/apt/lists/*

# Configure Antigravity auto-updates
RUN echo 'APT::Periodic::Update-Package-Lists "1";' > /etc/apt/apt.conf.d/20auto-upgrades \
  && echo 'APT::Periodic::Unattended-Upgrade "1";' >> /etc/apt/apt.conf.d/20auto-upgrades \
  && echo 'Unattended-Upgrade::Allowed-Origins { "antigravity-auto-updater-dev:antigravity-debian"; };' \
  > /etc/apt/apt.conf.d/50unattended-upgrades

# =============================================================================
# 6. Switch to dev user
# =============================================================================
USER dev
WORKDIR /home/dev

# =============================================================================
# 7. Install mise (polyglot runtime manager)
# =============================================================================
RUN curl -fsSL https://mise.run | sh
ENV PATH="/home/dev/.local/bin:${PATH}"

# Activate mise in bash (interactive + non-interactive)
RUN echo 'eval "$(mise activate bash)"' >> ~/.bashrc

# =============================================================================
# 8. Install Node.js via mise (needed for global npm tools)
# =============================================================================
RUN mise use --global node@22

# =============================================================================
# 9. Install Global AI & Dev Tools via npm (using mise's node)
# =============================================================================

# Claude Code (@anthropic-ai/claude-code)
RUN mise exec -- npm install -g @anthropic-ai/claude-code

# Gemini CLI (@google/gemini-cli)
RUN mise exec -- npm install -g @google/gemini-cli

# Chrome DevTools MCP Server
RUN mise exec -- npm install -g chrome-devtools-mcp

# =============================================================================
# 10. Configure Gemini CLI with chrome-devtools-mcp (headless=false)
# =============================================================================
RUN mkdir -p /home/dev/.gemini \
  && printf '{\n\
  "mcpServers": {\n\
  "chrome-devtools": {\n\
  "command": "chrome-devtools-mcp",\n\
  "args": [],\n\
  "env": {\n\
  "CHROME_PATH": "/usr/bin/google-chrome",\n\
  "HEADLESS": "false"\n\
  },\n\
  "trusted": true\n\
  }\n\
  }\n\
  }' > /home/dev/.gemini/settings.json

# =============================================================================
# 11. Configure Fluxbox Menu
# =============================================================================
RUN mkdir -p /home/dev/.fluxbox && \
    printf '[begin] (  Application Menu  )\n\
    [exec] (File Manager) { nautilus ~ } <>\n\
    [exec] (Text Editor) { mousepad } <>\n\
    [exec] (Terminal) { tilix -w ~ -e $(readlink -f /proc/$$/exe) -il } <>\n\
    [exec] (Web Browser) { x-www-browser --disable-dev-shm-usage } <>\n\
    [exec] (Antigravity) { tilix -t "Antigravity" -e antigravity } <>\n\
    [submenu] (System) {}\n\
        [exec] (Set Resolution) { tilix -t "Set Resolution" -e bash /usr/local/bin/set-resolution } <>\n\
        [exec] (Edit Application Menu) { mousepad ~/.fluxbox/menu } <>\n\
        [exec] (Passwords and Keys) { seahorse } <>\n\
        [exec] (Top Processes) { tilix -t "Top" -e htop } <>\n\
        [exec] (Disk Utilization) { tilix -t "Disk Utilization" -e ncdu / } <>\n\
        [exec] (Editres) {editres} <>\n\
        [exec] (Xfontsel) {xfontsel} <>\n\
        [exec] (Xkill) {xkill} <>\n\
        [exec] (Xrefresh) {xrefresh} <>\n\
    [end]\n\
    [config] (Configuration)\n\
    [workspaces] (Workspaces)\n\
[end]' > /home/dev/.fluxbox/menu



