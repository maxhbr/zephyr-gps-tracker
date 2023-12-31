ARG ZEPHYR_CI_IMAGE_VERSION="v0.26.4"

FROM ghcr.io/zephyrproject-rtos/zephyr-build:${ZEPHYR_CI_IMAGE_VERSION}

ARG NRF_COMMAND_LINE_TOOLS_URL="https://nsscprodmedia.blob.core.windows.net/prod/software-and-other-downloads/desktop-software/nrf-command-line-tools/sw/versions-10-x-x/10-22-1/nrf-command-line-tools_10.22.1_amd64.deb"
ARG SEGGER_JLINK_VERSION="V788l"
ARG UNIFLASH_SL_URL="https://dr-download.ti.com/software-development/software-programming-tool/MD-QeJBJLj8gq/8.3.0/uniflash_sl.8.3.0.4307.run"
ARG OPENOCD_URL="https://github.com/xpack-dev-tools/openocd-xpack/releases/download/v0.12.0-1/xpack-openocd-0.12.0-1-linux-arm64.tar.gz"

USER root

ARG USERNAME=user
ARG USER_UID=1000
ARG USER_GID=$USER_UID
COPY library-scripts/*.sh library-scripts/*.env /tmp/library-scripts/



RUN set -x \
    && apt-get update && export DEBIAN_FRONTEND=noninteractive \
    && /bin/bash /tmp/library-scripts/common-debian.sh "${INSTALL_ZSH}" "${USERNAME}" "${USER_UID}" "${USER_GID}" "${UPGRADE_PACKAGES}" "true" "true"\
    && usermod --shell "$(which zsh)" "${USERNAME}" \
    && usermod -a -G dialout "${USERNAME}" \

    && apt-get install -y \
            minicom picocom \
            wget \
            unzip \
            udev \
            libusb-1.0-0 \
            openocd \
    
    && apt-get autoremove -y && apt-get clean -y && rm -rf /var/lib/apt/lists/* /tmp/library-scripts

###########################################################################
##  start cc1352-flasher  #################################################
RUN set -x \
    && pip3 install pyserial intelhex python-magic \
    && git clone --depth 1 https://git.beagleboard.org/beagleconnect/cc1352-flasher /opt/cc1352-flasher \
    && ln -s /opt/cc1352-flasher/cc1352-flasher.py /usr/bin/cc1352-flasher.py \
    && ln -s /opt/cc1352-flasher/cc1352-flasher.py /usr/bin/cc1352-flasher
##  end cc1352-flasher  ###################################################
###########################################################################

###########################################################################
##  start nrfjprog  #######################################################
RUN set -x \
    && apt-get update && export DEBIAN_FRONTEND=noninteractive \

    && apt-get install -y udev \
	&& /lib/systemd/systemd-udevd --daemon manually \

	&& wget \
            -nv "$NRF_COMMAND_LINE_TOOLS_URL"\
            -O "/tmp/nrf-command-line-tools.deb" \
	&& ( dpkg -i "/tmp/nrf-command-line-tools.deb" || apt-get -f install -y || true) \
	&& rm "/tmp/nrf-command-line-tools.deb" \

	&& wget --post-data 'accept_license_agreement=accepted&submit=Download+software' \
			-nv "https://www.segger.com/downloads/jlink/JLink_Linux_${SEGGER_JLINK_VERSION}_x86_64.deb" \
			-O "/tmp/JLink_Linux_x86_64.deb" \
	&& ( dpkg -i "/tmp/JLink_Linux_x86_64.deb" || apt-get -f install -y || true ) \
	&& rm "/tmp/JLink_Linux_x86_64.deb" \

    && apt-get autoremove -y && apt-get clean -y && rm -rf /var/lib/apt/lists/* /tmp/library-scripts
    ##  end nrfjprog  #########################################################
    ###########################################################################

###########################################################################
##  start TI UniFlash  ####################################################
RUN set -x \
    && wget \
            -nv "$UNIFLASH_SL_URL" \
            -O "/opt/uniflash_sl.run" \
    && chmod +x "/opt/uniflash_sl.run"
    ## TODO: install uniflash
##  start TI UniFlash  ####################################################
###########################################################################

USER "${USERNAME}"
