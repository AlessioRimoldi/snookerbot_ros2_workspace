FROM ros:jazzy

ARG USERNAME=r
ARG USER_UID=1000
ARG USER_GID=$USER_UID

# 1) Clean up any existing user with the same UID
RUN if id -u $USER_UID 2>/dev/null; then userdel "$(id -un $USER_UID)"; fi

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
 && echo 'eval "$(starship init bash)"' >> /etc/bash.bashrc \
 && rm -rf /var/lib/apt/lists/*

# 3) Create group & user, give sudo rights
RUN groupadd --gid $USER_GID $USERNAME \
 && useradd --uid $USER_UID --gid $USER_GID -m $USERNAME \
 && echo "$USERNAME ALL=(root) NOPASSWD:ALL" > /etc/sudoers.d/$USERNAME \
 && chmod 0440 /etc/sudoers.d/$USERNAME

# 4) Ensure your workspace path exists and is owned by r
RUN mkdir -p /home/${USERNAME}/snookerbot_ros2_workspace \
 && chown -R ${USERNAME}:${USERNAME} /home/${USERNAME}/snookerbot_ros2_workspace

# 5) Ensure Bash remains the default shell
ENV SHELL=/bin/bash

# 6) Drop to user r and set working dir
USER $USERNAME
WORKDIR /home/$USERNAME/snookerbot_ros2_workspace

# 7) Add source setup.bash to .bashrc
RUN echo "source /opt/ros/jazzy/setup.bash" >> ~/.bashrc

CMD [ "bash" ]
