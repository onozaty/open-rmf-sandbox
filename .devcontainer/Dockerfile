FROM ubuntu:24.04

# ROS 2 Jazzy
# https://docs.ros.org/en/jazzy/Installation/Ubuntu-Install-Debs.html
# 
# Open RMF
# https://github.com/open-rmf/rmf?tab=readme-ov-file#setup

SHELL ["/bin/bash", "-c"]

# aptが安定するように、ミラーサイトを変更
RUN apt-get update && apt-get install -y ca-certificates
RUN sed -i.bak -r 's@http://(jp\.)?archive\.ubuntu\.com/ubuntu/?@https://ftp.udx.icscoe.jp/Linux/ubuntu/@g' /etc/apt/sources.list.d/ubuntu.sources

# System setup - Set locale
RUN apt-get update && apt-get install -y locales && \
    locale-gen en_US en_US.UTF-8 && \
    update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
ENV LANG=en_US.UTF-8

# Enable required repositories
RUN apt-get update && apt-get install -y \
    software-properties-common \
    curl && \
    add-apt-repository universe

# Add ROS 2 GPG key
RUN curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg

# Add repository to sources list
RUN echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" | tee /etc/apt/sources.list.d/ros2.list > /dev/null

# Install development tools
RUN apt-get update && apt-get install -y ros-dev-tools

# Update repository caches and upgrade system
RUN apt-get update && apt-get upgrade -y

# Install ROS 2 (Desktop Install - Recommended)
RUN apt-get install -y ros-jazzy-desktop

# Initialize rosdep
# https://docs.ros.org/en/jazzy/Installation/Alternatives/Ubuntu-Install-Binary.html#linux-install-binary-install-missing-dependencies
RUN rosdep init

# Install Open RMF
RUN apt-get update && apt-get install -y ros-jazzy-rmf-dev

# rmf_demosの準備
WORKDIR /opt/rmf_demos

# rmf_demosのjazzyブランチをzipでダウンロードしてビルド
RUN wget https://github.com/open-rmf/rmf_demos/archive/refs/heads/jazzy.zip -O rmf_demos.zip && \
    unzip rmf_demos.zip && \
    mkdir src && \
    mv rmf_demos-jazzy src/rmf_demos && \
    rm -rf rmf_demos.zip

RUN colcon mixin add default https://raw.githubusercontent.com/colcon/colcon-mixin-repository/master/index.yaml && colcon mixin update default

RUN source /opt/ros/jazzy/setup.bash && \
    source /usr/share/colcon_argcomplete/hook/colcon-argcomplete.bash && \
    rosdep update && \
    rosdep install --from-paths src --ignore-src -r -y && \
    colcon build

# ビルド時に ~/.gazebo/models にダウンロードされたモデルを ubuntu ユーザーのホームディレクトリにコピー
RUN cp -r /root/.gazebo /home/ubuntu/ && \
    chown -R ubuntu:ubuntu /home/ubuntu/.gazebo

RUN apt-get update && apt-get install -y ros-jazzy-ros-gz

# ロボットのModelのパスが正しく設定されていないため、シンボリックリンクを作成
RUN cd /opt/rmf_demos/install/rmf_demos_assets/share/rmf_demos_assets/models/ && \
    mkdir Open-RMF && \
    cd Open-RMF && \
    ln -s ../Caddy ./Caddy && \
    ln -s ../CleanerBotA ./CleanerBotA && \
    ln -s ../CleanerBotE ./CleanerBotE && \
    ln -s ../DeliveryRobot ./DeliveryRobot && \
    ln -s ../HospitalRobot ./HospitalRobot && \
    ln -s ../RobotPlaceholder ./RobotPlaceholder && \
    ln -s ../TeleportDispenser ./TeleportDispenser && \
    ln -s ../TeleportIngestor ./TeleportIngestor && \
    ln -s ../TinyRobot ./TinyRobot

# その他の必要なパッケージをインストール
RUN apt-get install -y \
    vim \
    # GUI関連 (X11フォワーディング用)
    x11-apps

# sudoでのパスワードなしでの実行を許可するための設定
RUN echo 'ubuntu ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers

# ユーザー切り替え
USER ubuntu

# Setup environment
RUN echo "source /opt/ros/jazzy/setup.bash" >> ~/.bashrc && \
    echo "source /usr/share/colcon_argcomplete/hook/colcon-argcomplete.bash" >> ~/.bashrc && \
    echo "source /opt/rmf_demos/install/setup.bash" >> ~/.bashrc

# system pluginのパスを設定
RUN echo "export GZ_SIM_SYSTEM_PLUGIN_PATH=/opt/ros/jazzy/lib/rmf_robot_sim_gz_plugins:\$GZ_SIM_SYSTEM_PLUGIN_PATH" >> ~/.bashrc && \
    echo "export GZ_SIM_SYSTEM_PLUGIN_PATH=/opt/ros/jazzy/lib/rmf_building_sim_gz_plugins:\$GZ_SIM_SYSTEM_PLUGIN_PATH" >> ~/.bashrc
# gui pluginのパスを設定
RUN echo "export GZ_GUI_PLUGIN_PATH=/opt/ros/jazzy/lib/rmf_building_sim_gz_plugins:\$GZ_GUI_PLUGIN_PATH" >> ~/.bashrc
