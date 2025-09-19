FROM osrf/ros:jazzy-desktop

# 2) Install curl, starship, sudo, pip
RUN apt-get update \
 && apt-get install -y --no-install-recommends \
      curl \
      nano \
      tmux \
      openssh-client \
      openssh-server \
      sudo \
      python3-pip \
      ros-jazzy-desktop \
      ros-jazzy-rviz-ogre-vendor \ 
      x11-apps \   
#  && sh -c "$(curl -fsSL https://starship.rs/install.sh)" -- -y \                
#  && echo 'eval "$(starship init bash)"' >> /etc/bash.bashrc \
 && rm -rf /var/lib/apt/lists/*

# --- Create dev user 'r' with passwordless sudo (needed by postCreateCommand) ---
ARG USERNAME=r
ARG USER_UID=1000
ARG USER_GID=${USER_UID}

USER root

# # deps for repo setup & GUI sanity checks
# RUN apt-get update && apt-get install -y --no-install-recommends \
#       wget gnupg lsb-release ca-certificates mesa-utils \
#   && rm -rf /var/lib/apt/lists/*

# # Add OSRF Gazebo (gz) repository
# RUN set -eux; \
#     wget -O /usr/share/keyrings/gazebo-archive-keyring.gpg https://packages.osrfoundation.org/gazebo.gpg; \
#     echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/gazebo-archive-keyring.gpg] http://packages.osrfoundation.org/gazebo/ubuntu $(lsb_release -cs) stable" \
#       > /etc/apt/sources.list.d/gazebo-stable.list

# # Install Gazebo Harmonic + tools and ROS 2 Jazzy bridge
# RUN apt-get update && apt-get install -y --no-install-recommends \
#       gz-sim7 gz-gui7 gz-tools \
#       ros-jazzy-ros-gz ros-jazzy-ros-gz-bridge \
#   && rm -rf /var/lib/apt/lists/*

# # (Optional) convenience: allow gz to find resources added later under ~/gz_resources
# RUN mkdir -p /home/r/gz_resources \
#  && chown -R r:r /home/r/gz_resources

# RUN set -eux; \
#     # Ensure the group ${USER_GID} exists and is named ${USERNAME}
#     if getent group "${USER_GID}" >/dev/null; then \
#         EXISTING_GROUP="$(getent group ${USER_GID} | cut -d: -f1)"; \
#         if [ "${EXISTING_GROUP}" != "${USERNAME}" ]; then \
#             groupmod -n "${USERNAME}" "${EXISTING_GROUP}"; \
#         fi; \
#     else \
#         groupadd --gid "${USER_GID}" "${USERNAME}"; \
#     fi; \
#     # Ensure the user at UID ${USER_UID} exists and is named ${USERNAME}
#     if getent passwd "${USER_UID}" >/dev/null; then \
#         EXISTING_USER="$(getent passwd ${USER_UID} | cut -d: -f1)"; \
#         if [ "${EXISTING_USER}" != "${USERNAME}" ]; then \
#             usermod -l "${USERNAME}" -d "/home/${USERNAME}" -m "${EXISTING_USER}"; \
#         else \
#             # user has correct name but wrong home? ensure home exists
#             mkdir -p "/home/${USERNAME}"; chown "${USER_UID}:${USER_GID}" "/home/${USERNAME}"; \
#         fi; \
#     elif id -u "${USERNAME}" >/dev/null 2>&1; then \
#         # user exists with different UID → remap UID
#         usermod -u "${USER_UID}" "${USERNAME}"; \
#     else \
#         useradd --uid "${USER_UID}" --gid "${USER_GID}" -m -s /bin/bash "${USERNAME}"; \
#     fi; \
#     apt-get update && apt-get install -y --no-install-recommends sudo && \
#     echo "${USERNAME} ALL=(ALL) NOPASSWD:ALL" >/etc/sudoers.d/${USERNAME} && \
#     chmod 0440 /etc/sudoers.d/${USERNAME} && \
#     rm -rf /var/lib/apt/lists/*

# Source ROS in system bashrc (affects all interactive bash shells)
RUN echo 'if [ -f /opt/ros/jazzy/setup.bash ]; then source /opt/ros/jazzy/setup.bash; fi' >> /etc/bash.bashrc

# Source ROS for login shells too (some env probes use login shells)
RUN printf '%s\n' \
  '#!/bin/sh' \
  'if [ -f /opt/ros/jazzy/setup.bash ]; then' \
  '  . /opt/ros/jazzy/setup.bash' \
  'fi' \
  > /etc/profile.d/ros.sh && chmod 0644 /etc/profile.d/ros.sh

RUN su - ${USERNAME} -c 'echo "if [ -f /opt/ros/jazzy/setup.bash ]; then source /opt/ros/jazzy/setup.bash; fi" >> ~/.bashrc'

# 4) Ensure your workspace path exists and is owned by r
RUN mkdir -p /home/${USERNAME}/snookerbot_ros2_workspace \
 && chown -R ${USERNAME}:${USERNAME} /home/${USERNAME}/snookerbot_ros2_workspace

# 5) Ensure Bash remains the default shell
ENV SHELL=/bin/bash


# 6) Drop to user r and set working dir
USER $USERNAME
WORKDIR /home/$USERNAME/snookerbot_ros2_workspace


CMD [ "bash" ]
