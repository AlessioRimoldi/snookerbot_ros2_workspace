# --- final stage ---
FROM osrf/ros:jazzy-desktop

# 0) Make sure ARGs are (re)declared in this stage
ARG USERNAME=r
ARG USER_UID=1000
ARG USER_GID=${USER_UID}

# 1) Create/normalize the user FIRST (idempotent even if UID/GID exist)
USER root
RUN set -eux; \
    if getent group "${USER_GID}" >/dev/null; then \
      g="$(getent group ${USER_GID} | cut -d: -f1)"; [ "$g" = "${USERNAME}" ] || groupmod -n "${USERNAME}" "$g"; \
    else groupadd -g "${USER_GID}" "${USERNAME}"; fi; \
    if getent passwd "${USER_UID}" >/dev/null; then \
      u="$(getent passwd ${USER_UID} | cut -d: -f1)"; [ "$u" = "${USERNAME}" ] || usermod -l "${USERNAME}" -d "/home/${USERNAME}" -m "$u"; \
    elif id -u "${USERNAME}" >/dev/null 2>&1; then usermod -u "${USER_UID}" "${USERNAME}"; \
    else useradd -u "${USER_UID}" -g "${USER_GID}" -m -s /bin/bash "${USERNAME}"; fi; \
    apt-get update && apt-get install -y --no-install-recommends sudo && \
    echo "${USERNAME} ALL=(ALL) NOPASSWD:ALL" >/etc/sudoers.d/${USERNAME} && chmod 0440 /etc/sudoers.d/${USERNAME} && \
    rm -rf /var/lib/apt/lists/*

# 2) Add ROS sourcing (system-wide + login shells)
RUN echo 'if [ -f /opt/ros/jazzy/setup.bash ]; then source /opt/ros/jazzy/setup.bash; fi' >> /etc/bash.bashrc
RUN printf '%s\n' \
  '#!/bin/sh' \
  'if [ -f /opt/ros/jazzy/setup.bash ]; then' \
  '  . /opt/ros/jazzy/setup.bash' \
  'fi' > /etc/profile.d/ros.sh && chmod 0644 /etc/profile.d/ros.sh

# 3) Append to the *user’s* ~/.bashrc (now that the user exists)
RUN runuser -u ${USERNAME} -- bash -lc 'echo "if [ -f /opt/ros/jazzy/setup.bash ]; then source /opt/ros/jazzy/setup.bash; fi" >> ~/.bashrc'

####################################### Adding GAZEBO
RUN apt-get update && apt-get install -y --no-install-recommends \
      wget curl gnupg ca-certificates lsb-release mesa-utils \
 && rm -rf /var/lib/apt/lists/*

# OSRF Gazebo repo (per official docs)
RUN set -eux; \
  curl -fsSL https://packages.osrfoundation.org/gazebo.gpg \
    -o /usr/share/keyrings/pkgs-osrf-archive-keyring.gpg; \
  echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/pkgs-osrf-archive-keyring.gpg] \
https://packages.osrfoundation.org/gazebo/ubuntu-stable $(lsb_release -cs) main" \
    > /etc/apt/sources.list.d/gazebo-stable.list

# Install Gazebo Ionic metapackage (brings gz sim/gui/tools) + ROS ↔ Gazebo bridge
RUN apt-get update && apt-get install -y --no-install-recommends \
      gz-ionic \
      ros-jazzy-ros-gz ros-jazzy-ros-gz-bridge \
 && rm -rf /var/lib/apt/lists/*

# (optional) a place for your custom gz resources
RUN mkdir -p /home/r/gz_resources && chown -R r:r /home/r/gz_resources
########################################

# 4) Workspace path after user exists
RUN mkdir -p /home/${USERNAME}/snookerbot_ros2_workspace && chown -R ${USER_UID}:${USER_GID} /home/${USERNAME}
WORKDIR /home/${USERNAME}/snookerbot_ros2_workspace
USER ${USERNAME}
