#### Build Stage ####
ARG BASE_IMAGE="ubuntu:20.04"
FROM $BASE_IMAGE AS base_layer

### Environment config
ARG BG_IMG=bg_kasm.png
ARG EXTRA_SH=noop.sh
ARG DISTRO=ubuntu
ARG LANG='en_US.UTF-8'
ARG LANGUAGE='en_US:en'
ARG LC_ALL='en_US.UTF-8'
ENV DEBIAN_FRONTEND=noninteractive \
    DISTRO=$DISTRO \
    HOME=/home/kasm-default-profile \
    INST_SCRIPTS=/dockerstartup/install \
    KASM_VNC_PATH=/usr/share/kasmvnc \
    LANG=$LANG \
    LANGUAGE=$LANGUAGE \
    LC_ALL=$LC_ALL \
    STARTUPDIR=/dockerstartup

### Home setup
WORKDIR $HOME
RUN mkdir -p $HOME/Desktop

### Support NVIDIA gpus for graphics acceleration
RUN echo "/usr/local/nvidia/lib" >> /etc/ld.so.conf.d/nvidia.conf && \
    echo "/usr/local/nvidia/lib64" >> /etc/ld.so.conf.d/nvidia.conf
COPY ./ubuntu/install/nvidia/10_nvidia.json /usr/share/glvnd/egl_vendor.d/10_nvidia.json

### Setup package rules
COPY ./ubuntu/install/package_rules $INST_SCRIPTS/package_rules/
RUN bash $INST_SCRIPTS/package_rules/package_rules.sh && rm -rf $INST_SCRIPTS/package_rules/

### Install common tools
COPY ./ubuntu/install/tools $INST_SCRIPTS/tools/
RUN bash $INST_SCRIPTS/tools/install_tools.sh && rm -rf $INST_SCRIPTS/tools/

### Copy over the maximization script to our startup dir for use by app images.
COPY ./ubuntu/install/maximize_script $STARTUPDIR/

### Install custom fonts
COPY ./ubuntu/install/fonts $INST_SCRIPTS/fonts/
RUN bash $INST_SCRIPTS/fonts/install_custom_fonts.sh && rm -rf $INST_SCRIPTS/fonts/

### Install xfce UI
COPY ./ubuntu/install/xfce $INST_SCRIPTS/xfce/
RUN bash $INST_SCRIPTS/xfce/install_xfce_ui.sh && rm -rf $INST_SCRIPTS/xfce/
ADD ./$DISTRO/xfce/.config/ $HOME/.config/
RUN mkdir -p /usr/share/extra/backgrounds/
RUN mkdir -p /usr/share/extra/icons/
ADD ./common/resources/images/bg_kasm.png  /usr/share/extra/backgrounds/bg_kasm.png
ADD ./common/resources/images/$BG_IMG  /usr/share/extra/backgrounds/bg_default.png
ADD ./common/resources/images/icon_ubuntu.png /usr/share/extra/icons/icon_ubuntu.png
ADD ./common/resources/images/icon_ubuntu.png /usr/share/extra/icons/icon_default.png
ADD ./common/resources/images/icon_kasm.png /usr/share/extra/icons/icon_kasm.png

### Install kasm_vnc dependencies and binaries
COPY ./ubuntu/install/kasm_vnc $INST_SCRIPTS/kasm_vnc/
RUN bash $INST_SCRIPTS/kasm_vnc/install_kasm_vnc.sh && rm -rf $INST_SCRIPTS/kasm_vnc/
COPY ./common/install/kasm_vnc/kasmvnc.yaml /etc/kasmvnc/

### Install Kasm Upload Server
#COPY ./ubuntu/install/kasm_upload_server $INST_SCRIPTS/kasm_upload_server/
#RUN bash $INST_SCRIPTS/kasm_upload_server/install_kasm_upload_server.sh  && rm -rf $INST_SCRIPTS/kasm_upload_server/

### Install Audio
COPY ./ubuntu/install/audio $INST_SCRIPTS/audio/
RUN bash $INST_SCRIPTS/audio/install_audio.sh  && rm -rf $INST_SCRIPTS/audio/

### Install Audio Input
#COPY ./ubuntu/install/audio_input $INST_SCRIPTS/audio_input/
#RUN bash $INST_SCRIPTS/audio_input/install_audio_input.sh && rm -rf $INST_SCRIPTS/audio_input/

### Install Gamepad Service
# COPY ./ubuntu/install/gamepad $INST_SCRIPTS/gamepad/
# RUN bash $INST_SCRIPTS/gamepad/install_gamepad.sh && rm -rf $INST_SCRIPTS/gamepad/

### Install custom cursors
COPY ./ubuntu/install/cursors $INST_SCRIPTS/cursors/
RUN bash $INST_SCRIPTS/cursors/install_cursors.sh && rm -rf $INST_SCRIPTS/cursors/

### Install Squid
COPY ./ubuntu/install/squid/install/ $INST_SCRIPTS/squid_install/
RUN bash $INST_SCRIPTS/squid_install/install_squid.sh && rm -rf $INST_SCRIPTS/squid_install/
COPY ./ubuntu/install/squid/resources/*.conf /etc/squid/
COPY ./ubuntu/install/squid/resources/start_squid.sh /etc/squid/start_squid.sh
COPY ./ubuntu/install/squid/resources/SN.png /usr/local/squid/share/icons/SN.png
RUN chown proxy:proxy /usr/local/squid/share/icons/SN.png
COPY ./ubuntu/install/squid/resources/error_message/access_denied.html /usr/local/squid/share/errors/en/ERR_ACCESS_DENIED
RUN chown proxy:proxy /usr/local/squid/share/errors/en/ERR_ACCESS_DENIED
RUN rm -rf $INST_SCRIPTS/resources/
RUN chmod +x /etc/squid/kasm_squid_adapter
RUN chmod +x /etc/squid/start_squid.sh && chmod 4755 /etc/squid/start_squid.sh

### configure startup
COPY ./common/scripts/kasm_hook_scripts $STARTUPDIR
ADD ./common/startup_scripts $STARTUPDIR
RUN bash $STARTUPDIR/set_user_permission.sh $STARTUPDIR $HOME && \
    echo 'source $STARTUPDIR/generate_container_user.sh' >> $HOME/.bashrc

### extra configurations needed per distro variant
COPY ./ubuntu/install/extra $INST_SCRIPTS/extra/
RUN bash $INST_SCRIPTS/extra/$EXTRA_SH  && rm -rf $INST_SCRIPTS/extra/

### VirtualGL
COPY ./ubuntu/install/virtualgl $INST_SCRIPTS/virtualgl/
RUN bash $INST_SCRIPTS/virtualgl/install_virtualgl.sh && rm -rf $INST_SCRIPTS/virtualgl/

### Create user and home directory for base images that don't already define it
RUN (groupadd -g 1000 kasm-user \
    && useradd -M -u 1000 -g 1000 kasm-user \
    && usermod -a -G kasm-user kasm-user) ; exit 0
ENV HOME=/home/kasm-user
WORKDIR $HOME
RUN mkdir -p $HOME && chown -R 1000:0 $HOME

### FIX PERMISSIONS ## Objective is to change the owner of non-home paths to root, remove write permissions, and set execute where required
# these files are created on container first exec, by the default user, so we have to create them since default will not have write perm
RUN touch $STARTUPDIR/wm.log \
    && touch $STARTUPDIR/window_manager_startup.log \
    && touch $STARTUPDIR/vnc_startup.log \
    && touch $STARTUPDIR/no_vnc_startup.log \
    && chown -R root:root $STARTUPDIR \
    && find $STARTUPDIR -type d -exec chmod 755 {} \; \
    && find $STARTUPDIR -type f -exec chmod 644 {} \; \
    && find $STARTUPDIR -type f -iname "*.sh" -exec chmod 755 {} \; \
    && find $STARTUPDIR -type f -iname "*.py" -exec chmod 755 {} \; \
    && find $STARTUPDIR -type f -iname "*.rb" -exec chmod 755 {} \; \
    && find $STARTUPDIR -type f -iname "*.pl" -exec chmod 755 {} \; \
    && find $STARTUPDIR -type f -iname "*.log" -exec chmod 666 {} \; \
    # && chmod 755 $STARTUPDIR/upload_server/kasm_upload_server \
    # && chmod 755 $STARTUPDIR/audio_input/kasm_audio_input_server \
    # && chmod 755 $STARTUPDIR/gamepad/kasm_gamepad_server \
    && chmod 755 $STARTUPDIR/generate_container_user.sh \
    && chmod +x $STARTUPDIR/jsmpeg/kasm_audio_out-linux \
    && rm -rf $STARTUPDIR/install \
    # && mkdir -p $STARTUPDIR/kasmrx/Downloads \
    # && chown 1000:1000 $STARTUPDIR/kasmrx/Downloads \
    && chown -R root:root /usr/local/bin \
    && chown 1000:root /var/run/pulse \
    && rm -Rf /home/kasm-default-profile/.launchpadlib

### Cleanup job
COPY ./ubuntu/install/cleanup $INST_SCRIPTS/cleanup/
RUN bash $INST_SCRIPTS/cleanup/cleanup.sh && rm -rf $INST_SCRIPTS/cleanup/

### Apply KasmVNC web UI fix
WORKDIR $HOME
RUN git clone --depth 1 --branch v1.3.0 https://github.com/kasmtech/noVNC.git

WORKDIR $HOME/noVNC
COPY ./common/install/kasm_vnc/www/app/ui.js $HOME/noVNC/app/ui.js
COPY ./common/install/kasm_vnc/www/package.json $HOME/noVNC/package.json
COPY ./common/install/kasm_vnc/www/vnc.html $HOME/noVNC/vnc.html
RUN curl -fsSL https://deb.nodesource.com/setup_20.x | bash - && apt-get install -y nodejs
RUN rm -rf package-lock.json \
    && npm i \
    && npm run build \
    && rm -rf /usr/share/kasmvnc/www/dist \
    && mv dist /usr/share/kasmvnc/www/ \
    && mv -f index.html /usr/share/kasmvnc/www/index.html

WORKDIR $HOME
RUN rm -rf noVNC

#### Runtime Stage ####
FROM scratch
COPY --from=base_layer / /

### Environment config
ARG DISTRO=ubuntu
ARG EXTRA_SH=noop.sh
ARG LANG='en_US.UTF-8'
ARG LANGUAGE='en_US:en'
ARG LC_ALL='en_US.UTF-8'
ARG START_PULSEAUDIO=1
ARG START_XFCE4=1
ENV AUDIO_PORT=4901 \
    DEBIAN_FRONTEND=noninteractive \
    DISPLAY=:1 \
    DISTRO=$DISTRO \
    GOMP_SPINCOUNT=0 \
    HOME=/home/kasm-user \
    INST_SCRIPTS=/dockerstartup/install \
    KASMVNC_AUTO_RECOVER=true \
    KASM_VNC_PATH=/usr/share/kasmvnc \
    LANG=$LANG \
    LANGUAGE=$LANGUAGE \
    LC_ALL=$LC_ALL \
    LD_LIBRARY_PATH=/opt/libjpeg-turbo/lib64/:/usr/local/lib/ \
    LD_LIBRARY_PATH=/usr/lib/x86_64-linux-gnu:/usr/lib/i386-linux-gnu${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}:/usr/local/nvidia/lib:/usr/local/nvidia/lib64 \
    MAX_FRAME_RATE=24 \
    NVIDIA_DRIVER_CAPABILITIES=${NVIDIA_DRIVER_CAPABILITIES:+$NVIDIA_DRIVER_CAPABILITIES,}graphics,compat32,utility \
    OMP_WAIT_POLICY=PASSIVE \
    PULSE_RUNTIME_PATH=/var/run/pulse \
    SDL_GAMECONTROLLERCONFIG="030000005e040000be02000014010000,XInput Controller,platform:Linux,a:b0,b:b1,x:b2,y:b3,back:b8,guide:b16,start:b9,leftstick:b10,rightstick:b11,leftshoulder:b4,rightshoulder:b5,dpup:b12,dpdown:b13,dpleft:b14,dpright:b15,leftx:a0,lefty:a1,rightx:a2,righty:a3,lefttrigger:b6,righttrigger:b7" \
    SHELL=/bin/bash \
    START_PULSEAUDIO=$START_PULSEAUDIO \
    STARTUPDIR=/dockerstartup \
    START_XFCE4=$START_XFCE4 \
    TERM=xterm \
    VNC_COL_DEPTH=24 \
    VNCOPTIONS="-PreferBandwidth -DynamicQualityMin=4 -DynamicQualityMax=7 -DLP_ClipDelay=0" \
    VNC_PORT=6901 \
    VNC_PW=vncpassword \
    VNC_DISABLE_AUTH=false \
    VNC_ALLOW_CLIENT_TO_OVERRIDE_VNC_SETTINGS=false \
    VNC_RESOLUTION=1280x1024 \
    VNC_RESOLUTION=1280x720 \
    VNC_VIEW_ONLY_PW=vncviewonlypassword \
    KASM_SVC_AUDIO=0 \
    KASM_SVC_AUDIO_INPUT=0 \
    KASM_SVC_UPLOADS=0 \
    KASM_SVC_GAMEPAD=0

### Ports and user
EXPOSE $VNC_PORT \
       $UPLOAD_PORT \
       $AUDIO_PORT
WORKDIR $HOME
USER 1000

ENTRYPOINT ["/dockerstartup/kasm_default_profile.sh", "/dockerstartup/vnc_startup.sh"]
CMD ["--wait"]
