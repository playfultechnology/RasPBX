#!/bin/bash

trap '{ stty sane; printf "\n\nAborting\n\n"; exit 1; }' SIGINT SIGTERM

pistatus()
{
  local STATUS=0

  STATUS=$(vcgencmd get_throttled | sed -n 's|^throttled=\(.*\)|\1|p')
  if [[ ${STATUS} -ne 0 ]]; then
    if [ $((${STATUS} & 0x00001)) -ne 0 ]; then
      echo "Power is currently Under Voltage"
    elif [ $((${STATUS} & 0x10000)) -ne 0 ]; then
      echo "Power has previously been Under Voltage"
    fi
    if [ $((${STATUS} & 0x00002)) -ne 0 ]; then
      echo "ARM Frequency is currently Capped"
    elif [ $((${STATUS} & 0x20000)) -ne 0 ]; then
      echo "ARM Frequency has previously been Capped"
    fi
    if [ $((${STATUS} & 0x00004)) -ne 0 ]; then
      echo "CPU is currently Throttled"
    elif [ $((${STATUS} & 0x40000)) -ne 0 ]; then
      echo "CPU has previously been Throttled"
    fi
    if [ $((${STATUS} & 0x00008)) -ne 0 ]; then
      echo "Currently at Soft Temperature Limit"
    elif [ $((${STATUS} & 0x80000)) -ne 0 ]; then
      echo "Previously at Soft Temperature Limit"
    fi
    echo ""
  fi
}

# Ensure root User
if [ $(id -u) -ne 0 ]; then
  echo ""
  echo "Must be run as root user: sudo $0"
  echo ""
  exit 1
fi

# Delete Profile Script
if [ -f /etc/profile.d/install-ps.sh ]; then
  rm /etc/profile.d/install-ps.sh
fi

# Check Operating System
OSVER="$(sed -n 's|^VERSION=".*(\(.*\))"|\1|p' /etc/os-release)"
if [[ "${OSVER}" != "bullseye" && "${OSVER}" != "bookworm" ]]; then
  echo ""
  echo "Unsupported operating system: ${OSVER}"
  echo ""
  exit 1
fi

# Enable PuTTY Logging
if [[ "$1" = "--phase2" || "$1" = "--phase3" ]]; then
  echo ""
  read -r -n 1 -s -p "Enable PuTTY logging and press any key to continue "
  echo ""
  echo ""
  echo -n "Start Time: "
  date
  echo ""
fi

# Display Storage Usage
echo ""
df
echo ""

# Wait For Time Synchronization
COUNT=0
while [ ! -f "/run/systemd/timesync/synchronized" ]
do
  echo -n "."
  (( COUNT += 1 ))
  sleep 1
done
while [ ${COUNT} -ne 0 ]
do
  echo -n -e "\b \b"
  (( COUNT -= 1 ))
done

# Wait For apt To Finish
while :
do
  ps -A | grep apt &> /dev/null
  if [ $? -ne 0 ]; then
    break
  else
    sleep 1
  fi
done
apt-get -y --allow-releaseinfo-change update

# Get Program Path
PGMPATH=$(realpath $0)

# Get BOOT partition mount point
BOOTMNT="$(sed -n 's|^\S\+\s\+\(/boot\S*\)\s\+.*$|\1|p' /etc/fstab)"

echo ""
echo ""
if [ "$1" = "" ]; then
  echo "FreePBX Installation: Phase 1"
  echo ""

  # Extract Installation Archive
  if [ ! -f ${PGMPATH}.tar.gz ]; then
    echo ""
    echo "${PGMPATH}.tar.gz is missing"
    echo ""
    exit 1
  fi
  tar xfz ${PGMPATH}.tar.gz -C /root
  chmod 755 /root/*

  # Set user Password
  if [ "${SUDO_USER}" != "" ]; then
    echo ""
    echo "Set ${SUDO_USER} User Password:"
    echo ""
    passwd "${SUDO_USER}"
  fi

  if [ $(tail -n 1 /etc/profile | grep -c echo) -eq 0 ]; then
    echo "echo" >> /etc/profile
  fi

  # Set root Password
  echo ""
  echo "Set root User Password:"
  echo ""
  passwd root

  # Enable SSH Login For root User
  sed -i 's/^#PermitRootLogin prohibit-password$/PermitRootLogin yes/' /etc/ssh/sshd_config
  systemctl restart ssh

  echo ""
  echo -e -n "a) 15.0\nb) 16.0\nc) 17.0\nFreePBX Version? "
  while read -r -n 1 -s answer; do
    if [[ ${answer} = [aAbBcC] ]]; then
      echo "${answer}"
      if [[ ${answer} = [aA] ]]; then
        VER_FREEPBX=15.0
      elif [[ ${answer} = [bB] ]]; then
        VER_FREEPBX=16.0
      elif [[ ${answer} = [cC] ]]; then
        VER_FREEPBX=17.0
      fi
      break
    fi
  done

  echo ""
  echo -e -n "a) 18\nb) 20\nc) 21\nd) 22\nAsterisk Version? "
  while read -r -n 1 -s answer; do
    if [[ ${answer} = [aAbBcCdD] ]]; then
      if [[ ${answer} = [aA] ]]; then
        echo "${answer}"
        VER_ASTERISK=18
      elif [[ ${answer} = [bB] ]]; then
        echo "${answer}"
        VER_ASTERISK=20
      elif [[ ${answer} = [cC] ]]; then
        if [ "${VER_FREEPBX}" = "17.0" ]; then
          echo "${answer}"
          VER_ASTERISK=21
        else
          continue
        fi
      elif [[ ${answer} = [dD] ]]; then
        if [ "${VER_FREEPBX}" = "17.0" ]; then
          echo "${answer}"
          VER_ASTERISK=22
        else
          continue
        fi
      fi
      break
    fi
  done

  echo ""
  echo -n "Enable Edge (y/n)? "
  while read -r -n 1 -s answer; do
    if [[ ${answer} = [yYnN] ]]; then
      echo "${answer}"
      if [[ ${answer} = [yY] ]]; then
        EDGE=TRUE
        edgeyn=Yes
      else
        EDGE=FALSE
        edgeyn=No
      fi
      break
    fi
  done

  echo ""
  echo -n "Disable IPv6 (y/n)? "
  while read -r -n 1 -s answer; do
    if [[ ${answer} = [yYnN] ]]; then
      echo "${answer}"
      if [[ ${answer} = [yY] ]]; then
        IPV6=FALSE
        ipv6yn=No
      else
        IPV6=TRUE
        ipv6yn=Yes
      fi
      break
    fi
  done

  echo ""
  echo "FreePBX Version: ${VER_FREEPBX}"
  echo "Asterisk Version: ${VER_ASTERISK}"
  echo "Edge Enabled: ${edgeyn}"
  echo "IPv6 Enabled: ${ipv6yn}"

  echo ""
  echo -n "Continue (y/n)? "
  while read -r -n 1 -s answer; do
    if [[ ${answer} = [yYnN] ]]; then
      echo "${answer}"
      if [[ ${answer} = [yY] ]]; then
        break
      else
        echo ""
        echo "Aborted"
        echo ""
        exit 1
      fi
    fi
  done

  echo "VER_FREEPBX=${VER_FREEPBX}" > /root/install/parameters
  echo "VER_ASTERISK=${VER_ASTERISK}" >> /root/install/parameters
  echo "EDGE=${EDGE}" >> /root/install/parameters
  echo "IPV6=${IPV6}" >> /root/install/parameters

  # Create Profile Script
  cat <<EOF > /etc/profile.d/install-ps.sh
#!/bin/bash

ps cax | grep install-ps.sh > /dev/null
if [[ \$? -ne 0 && \$(id -u) -eq 0 ]]; then
  ${PGMPATH} --phase2
fi
EOF

  # Enable Pushbutton Shutdown/Startup
  if [ $(grep -c dtoverlay=gpio-shutdown "${BOOTMNT}/config.txt") -eq 0 ]; then
    sed -i '1s/^/dtoverlay=gpio-shutdown\n\n/' "${BOOTMNT}/config.txt"
  fi

  # Set GPU Memory To 32MB
  if [ $(grep -c gpu_mem= "${BOOTMNT}/config.txt") -eq 0 ]; then
    sed -i '1s/^/gpu_mem=32\n\n/' "${BOOTMNT}/config.txt"
  fi

  # Set Hostname / Set Localisations / Expand Filesystem
  if [ -f "/sys/firmware/devicetree/base/system/linux,revision" ]; then
    BDINFO="$(od -v -An -t x1 /sys/firmware/devicetree/base/system/linux,revision | tr -d ' \n')"
  elif grep -q Revision /proc/cpuinfo; then
    BDINFO="$(sed -n '/^Revision/s/^.*: \(.*\)/\1/p' < /proc/cpuinfo)"
  elif command -v vcgencmd > /dev/null; then
    BDINFO="$(vcgencmd otp_dump | grep '30:' | sed 's/.*://')"
  else
    errexit "Raspberry Pi board info not found"
  fi
  BDCHIP=$(((0x${BDINFO} >> 12) & 15))
  if [[ ${BDCHIP} = 3 || ${BDCHIP} = 4 ]]; then
    RPI_45=TRUE
  else
    RPI_45=FALSE
  fi
  ROOT_PART="$(mount | sed -n 's|^/dev/\(.*\) on / .*|\1|p')"
  if [ "${ROOT_PART}" = "mmcblk0p2" ]; then
    if [ "${RPI_45}" = "FALSE" ]; then
      sed -i '/^[[:space:]]*#/!s|root=\S\+\s|root=/dev/mmcblk0p2 |' "${BOOTMNT}/cmdline.txt"
      sed -i '/^[[:space:]]*#/!s|^PARTUUID=........-0|/dev/mmcblk0p|' /etc/fstab
    fi
    raspi-config
  else
    if [ -b /dev/mmcblk0 ]; then
      if [ "${RPI_45}" = "FALSE" ]; then
        sed -i "/^[[:space:]]*#/!s|^\S\+\(\s\+/boot\S*\s\+vfat\s\+.*\)$|/dev/mmcblk0p1\1|" /etc/fstab
      fi
    fi
    ROOT_DEV="$(sed 's/[0-9]\+$//' <<< "${ROOT_PART}")"
    cp /usr/bin/raspi-config /tmp/raspi-config-usb-tmp
    sed -i -E "s/mmcblk0p?/${ROOT_DEV}/" /tmp/raspi-config-usb-tmp
    sed -i 's|    resize2fs /dev/$ROOT_PART &&|    ROOT_DEV=\\$(findmnt / -o source -n) \&\&\n    resize2fs \\$ROOT_DEV \&\&|' /tmp/raspi-config-usb-tmp
    /tmp/raspi-config-usb-tmp
    rm /tmp/raspi-config-usb-tmp
  fi

  # Reboot
  pistatus
  shutdown -r now
elif [ "$1" = "--phase2" ]; then
  echo "FreePBX Installation: Phase 2"
  echo ""

  # Update/Upgrade Raspberry Pi OS
  echo ""
  echo "Updating/Upgrading Raspberry Pi OS"
  if [ -e /etc/apt/listchanges.conf ]; then
    sed -i 's/frontend=pager/frontend=text/' /etc/apt/listchanges.conf
  fi
  if [ ! -e /etc/apt/sources.list.d/php.list ]; then
    if [ "${OSVER}" != "bookworm" ]; then
      wget -q https://packages.sury.org/php/apt.gpg -O- | apt-key add - &> /dev/null
    else
      wget -qO- https://packages.sury.org/php/apt.gpg | tee /etc/apt/trusted.gpg.d/apt.gpg > /dev/null
    fi
    echo "deb https://packages.sury.org/php/ ${OSVER} main" > /etc/apt/sources.list.d/php.list
  fi
  apt-get -y update
  apt-get -y upgrade
  apt-get -y dist-upgrade
  if [ -e /etc/apt/listchanges.conf ]; then
    sed -i 's/frontend=text/frontend=pager/' /etc/apt/listchanges.conf
  fi
  echo "Updating/Upgrading completed"
  echo ""
  echo -n "Finish Time: "
  date
  echo ""

  # Create Profile Script
  cat <<EOF > /etc/profile.d/install-ps.sh
#!/bin/bash

ps cax | grep install-ps.sh > /dev/null
if [[ \$? -ne 0 && \$(id -u) -eq 0 ]]; then
  ${PGMPATH} --phase3
fi
EOF

  # Reboot
  pistatus
  shutdown -r now
elif [ "$1" = "--phase3" ]; then
  echo "FreePBX Installation: Phase 3"
  echo ""

  # Retrieve Parameters
  if [ ! -e /root/install/parameters ]; then
    echo ""
    echo "/root/install/parameters is missing"
    echo ""
    exit 1
  fi
  OIFS=${IFS}
  IFS='='
  while read var val; do
    eval ${var}=${val}
  done < /root/install/parameters
  IFS=${OIFS}
  if [ "${EDGE}" = "TRUE" ]; then
    edgeyn=Yes
  else
    edgeyn=No
  fi
  if [ "${IPV6}" = "TRUE" ]; then
    ipv6yn=Yes
  else
    ipv6yn=No
  fi

  # Display Paramters
  echo ""
  echo "FreePBX Version: ${VER_FREEPBX}"
  echo "Asterisk Version: ${VER_ASTERISK}"
  echo "Edge Enabled: ${edgeyn}"
  echo "IPv6 Enabled: ${ipv6yn}"

  # Confirm Installation
  echo ""
  echo -n "Install now (y/n)? "
  while read -r -n 1 -s answer; do
    if [[ ${answer} = [yYnN] ]]; then
      echo "${answer}"
      if [[ ${answer} = [yY] ]]; then
        break
      else
        echo ""
        echo "Aborted"
        echo ""
        exit 1
      fi
    fi
  done
  echo ""

  # Regenerate SSH Keys
  modinfo bcm2708-rng &> /dev/null
  if [ $? -eq 0 ]; then
    echo "BCM2708 H/W Random Number Generator (RNG) driver is installed"
    modprobe bcm2708-rng
    if [ $(grep -c bcm2708-rng /etc/modules) -eq 0 ]; then
      echo "bcm2708-rng" >> /etc/modules
    fi
  else
    echo "BCM2708 H/W Random Number Generator (RNG) driver is not installed"
  fi
  dpkg -s rng-tools &> /dev/null
  INSTALLED=$?  
  apt-get -y install rng-tools
  if [ ${INSTALLED} -ne 0 ]; then
    echo "Waiting while system entropy pool is replenished"
    sleep 15
  fi
  rm /etc/ssh/ssh_host_*
  dpkg-reconfigure openssh-server
  systemctl restart ssh

  # Patch /sbin/dphys-swapfile For f2fs
  if [ $(grep -c 'if \[ $? -eq 0 ]; then' /sbin/dphys-swapfile) -ne 0 ]; then
    cp /sbin/dphys-swapfile /sbin/dphys-swapfile_orig
    sed -i 's|#! /bin/sh|#!/bin/bash|' /sbin/dphys-swapfile
    sed -i 's|      if \[ $? -eq 0 ]; then|      if [[ $? -eq 0 \&\& "$(mount \| sed -n '\''s\|^.* on / type \\(\\S\\+\\) .*\|\\1\|p'\'')" != "f2fs" ]]; then|' /sbin/dphys-swapfile
  fi

  # Enlarge Swap File
  echo ""
  echo "Enlarging Swap File"
  sed -i 's/CONF_SWAPSIZE=100/CONF_SWAPSIZE=256/' /etc/dphys-swapfile
  /etc/init.d/dphys-swapfile stop
  /etc/init.d/dphys-swapfile start

  # Install Dependencies
  if [ "${VER_FREEPBX}" = "15.0" ]; then
    apt-get -y install php5.6 php5.6-curl php5.6-cli php5.6-mysql php5.6-gd php5.6-mbstring php5.6-xml php5.6-sqlite3
  elif [ "${VER_FREEPBX}" = "16.0" ]; then
    apt-get -y install php7.4 php7.4-curl php7.4-cli php7.4-mysql php7.4-gd php7.4-mbstring php7.4-xml php7.4-sqlite3
  elif [ "${VER_FREEPBX}" = "17.0" ]; then
    apt-get -y install php8.2 php8.2-curl php8.2-cli php8.2-mysql php8.2-gd php8.2-mbstring php8.2-xml php8.2-sqlite3
  fi
  mv /etc/apt/preferences.d/php-common.pref /root &> /dev/null
  apt-get -y install build-essential wget dnsutils openssh-server apache2 bison flex curl sox cmake gdisk npm\
 libncurses5-dev libssl-dev libxml2-dev libnewt-dev sqlite3 libsqlite3-dev pkg-config automake libtool bc nodejs\
 autoconf git unixodbc-dev uuid uuid-dev libasound2-dev libogg-dev libvorbis-dev libcurl4-openssl-dev file rsyslog\
 libicu-dev libical-dev libneon27-dev libsrtp2-dev libspandsp-dev subversion rsync ntfs-3g debconf-utils xmlstarlet\
 bluetooth libbluetooth-dev bluez bluez-tools dirmngr libedit-dev mpg123 php-pear mariadb-server mariadb-client

  # Install MariaDB ODBC Connector
  cd /usr/src
  git clone https://github.com/MariaDB/mariadb-connector-odbc.git
  cd mariadb-connector-odbc
  git checkout tags/3.2.2
  mkdir build
  cd build
  if [ "$(uname -m)" = "aarch64" ]; then
    DDM_DIR=/usr/lib/aarch64-linux-gnu
  else
    DDM_DIR=/usr/lib/arm-linux-gnueabihf
  fi
  cmake ../ -LH -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr/local -DWITH_SSL=OPENSSL\
 -DDM_DIR="${DDM_DIR}" -DCMAKE_C_FLAGS_RELEASE:STRING="-w"
  cmake --build . --config Release
  make install

  # Download And Install Asterisk
  cd /usr/src
  wget http://downloads.asterisk.org/pub/telephony/asterisk/asterisk-${VER_ASTERISK}-current.tar.gz
  tar xfz asterisk-${VER_ASTERISK}-current.tar.gz
  rm asterisk-${VER_ASTERISK}-current.tar.gz
  cd /usr/src/asterisk*
  sed -i 's|\(^#define DEVICE_FRAME_SIZE \).*|\164|' addons/chan_mobile.c
  contrib/scripts/get_mp3_source.sh
  ./configure --with-pjproject-bundled --with-jansson-bundled --with-bluetooth
  make menuselect.makeopts
  menuselect/menuselect --enable CORE-SOUNDS-EN-WAV --enable CORE-SOUNDS-EN-ULAW menuselect.makeopts
  menuselect/menuselect --enable CORE-SOUNDS-EN-GSM --enable CORE-SOUNDS-EN-G722 menuselect.makeopts
  menuselect/menuselect --enable EXTRA-SOUNDS-EN-WAV --enable EXTRA-SOUNDS-EN-ULAW menuselect.makeopts
  menuselect/menuselect --enable EXTRA-SOUNDS-EN-GSM --enable EXTRA-SOUNDS-EN-G722 menuselect.makeopts
  menuselect/menuselect --enable format_mp3 --enable chan_mobile --enable app_macro menuselect.makeopts
  make
  RESULT=$?
  if [ ${RESULT} -ne 0 ]; then
    exit ${RESULT}
  fi
  make install
  make config
  ldconfig
  update-rc.d -f asterisk remove

  # Create FreePBX Service
  cat <<EOF > /etc/systemd/system/freepbx.service
[Unit]
Description=FreePBX VoIP Server
After=mysql.service
 
[Service]
Type=oneshot
RemainAfterExit=yes
ExecStart=/usr/sbin/fwconsole start -q
ExecStop=/usr/sbin/fwconsole stop -q
 
[Install]
WantedBy=multi-user.target
EOF

  systemctl daemon-reload
  systemctl enable freepbx.service

  # Create Asterisk User And Set Ownership Permissions
  useradd -m asterisk
  chown asterisk:asterisk /var/run/asterisk
  chown -R asterisk:asterisk /etc/asterisk
  chown -R asterisk:asterisk /var/{lib,log,spool}/asterisk
  chown -R asterisk:asterisk /usr/lib/asterisk
  rm -rf /var/www/html

  # Allow Asterisk Access
  echo "asterisk ALL = NOPASSWD: /sbin/shutdown" >> /etc/sudoers
  echo "asterisk ALL = NOPASSWD: /sbin/reboot" >> /etc/sudoers

  # Modify Apache & PHP
  TIMEZONE="$(cat /etc/timezone)"
  PHPDIR="$(ls -d /etc/php/* | sed -n 's|^/etc/php/\(.*\)|\1|p')"
  cp /etc/php/${PHPDIR}/apache2/php.ini /etc/php/${PHPDIR}/apache2/php.ini_orig
  sed -i "s|.*date\.timezone =.*|date.timezone = \"$TIMEZONE\"|" /etc/php/${PHPDIR}/apache2/php.ini
  sed -i 's/\(^upload_max_filesize = \).*/\120M/' /etc/php/${PHPDIR}/apache2/php.ini
  sed -i 's/\(^memory_limit = \).*/\1256M/' /etc/php/${PHPDIR}/apache2/php.ini
  cp /etc/php/${PHPDIR}/cli/php.ini /etc/php/${PHPDIR}/cli/php.ini_orig
  sed -i "s|.*date\.timezone =.*|date.timezone = \"$TIMEZONE\"|" /etc/php/${PHPDIR}/cli/php.ini
  sed -i 's/\(^upload_max_filesize = \).*/\120M/' /etc/php/${PHPDIR}/cli/php.ini
  cp /etc/apache2/apache2.conf /etc/apache2/apache2.conf_orig
  sed -i 's/^\(User\|Group\).*/\1 asterisk/' /etc/apache2/apache2.conf
  sed -i 's_\(^ErrorLog \).*_\1/dev/null_' /etc/apache2/apache2.conf
  sed -i 's/AllowOverride None/AllowOverride All/' /etc/apache2/apache2.conf
  cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/000-default.conf_orig
  sed -i 's_\(ErrorLog \).*_\1/dev/null_' /etc/apache2/sites-available/000-default.conf
  sed -i 's_\(CustomLog \).*_\1/dev/null combined_' /etc/apache2/sites-available/000-default.conf
  cp /etc/apache2/envvars /etc/apache2/envvars_orig
  echo "" >> /etc/apache2/envvars
  echo "export PATH='/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin'" >> /etc/apache2/envvars
  a2enmod rewrite
  systemctl restart apache2

  # Configure ODBC
  cat <<EOF > /etc/odbcinst.ini
[MySQL]
Description = ODBC for MySQL (MariaDB)
Driver = /usr/local/lib/mariadb/libmaodbc.so
FileUsage = 1
EOF

  cat <<EOF > /etc/odbc.ini
[MySQL-asteriskcdrdb]
Description = MySQL connection to 'asteriskcdrdb' database
Driver = MySQL
Server = localhost
Database = asteriskcdrdb
Port = 3306
Socket = /var/run/mysqld/mysqld.sock
Option = 3
EOF

  # Disable STRICT_TRANS_TABLES in MariaDB
  cat <<EOF > /etc/mysql/mariadb.conf.d/60-freepbx.cnf
#
# * FreePBX-related settings
#
# * Disable "STRICT_TRANS_TABLES"
#

[mysqld]
sql_mode = "ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION"
EOF

  systemctl restart mariadb

  # Download And Install FreePBX
  touch /etc/asterisk/{modules,statsd,smdi}.conf
  cd /usr/src
  git clone --branch release/${VER_FREEPBX} --depth 1 https://github.com/FreePBX/framework.git
  mv framework freepbx
  cd /usr/src/freepbx
  ./start_asterisk start
  ./install --no-interaction
  if [ $? -ne 0 ]; then
    if [ "${VER_FREEPBX}" != "17.0" ]; then   # !!! TEMPORARY WORKAROUND !!!
      echo ""
      echo "FreePBX installation failed!"
      echo ""
      exit 1
    fi
  fi
  if [ "${EDGE}" = "TRUE" ]; then
    fwconsole setting MODULEADMINEDGE 1
  fi
  fwconsole ma downloadinstall framework
  if [ "${VER_FREEPBX}" != "17.0" ]; then
    fwconsole reload
  fi
  fwconsole ma downloadinstall cdr conferences customappsreg dashboard featurecodeadmin infoservices music pm2 sipsettings voicemail
  fwconsole ma downloadinstall announcement callforward callrecording callwaiting donotdisturb findmefollow parking ringgroups
  fwconsole ma downloadinstall asteriskinfo configedit disa hotelwakeup iaxsettings miscapps miscdests outroutemsg superfecta
  fwconsole reload
  ln -s /var/lib/asterisk/moh /var/lib/asterisk/mohmp3

  # Customize FreePBX settings
  cat <<EOF >> /etc/asterisk/freepbx_menu.conf
; This configuration file is used to rearrange the menu navigation in FreePBX.
; You must also enable the Advanced Setting: Use freepbx_menu.conf Configuration

[dahdichandids]
remove=yes

[wiki]
remove=yes
EOF

  chown asterisk:asterisk /etc/asterisk/freepbx_menu.conf
  fwconsole setting HTTPBINDADDRESS 127.0.0.1
  fwconsole setting HTTPTLSBINDADDRESS 127.0.0.1
  fwconsole setting DIAL_OPTIONS ""
  fwconsole setting TRUNK_OPTIONS R
  fwconsole setting ASTCONFAPP app_confbridge
  fwconsole setting RINGTIMER 30
  fwconsole setting USE_FREEPBX_MENU_CONF 1
  fwconsole setting AMPDISABLELOG 1
  fwconsole setting PHP_ERROR_HANDLER_OUTPUT off
  fwconsole setting BROWSER_STATS 0
  fwconsole setting FREEPBX_SYSTEM_IDENT "Raspberry Pi"
  fwconsole setting SYS_STATS_DISABLE 1
  fwconsole setting PM2DISABLELOG 1
  fwconsole modulesystemmanager update_every day
  fwconsole modulesystemmanager auto_module_updates emailonly
  fwconsole modulesystemmanager auto_module_security_updates emailonly
  fwconsole setting LAUNCH_AGI_AS_FASTAGI 1

  # Clear Notifications
  NOTIFICATIONS="$(fwconsole notification --list)"
  echo "${NOTIFICATIONS}"
  if [ $(grep -c "FW_TAMPERED" <<< "${NOTIFICATIONS}") -ne 0 ]; then
    fwconsole notification --delete freepbx FW_TAMPERED
  fi
  if [ $(grep -c "EXTENSIONS_MOVE" <<< "${NOTIFICATIONS}") -ne 0 ]; then
    fwconsole notification --delete core EXTENSIONS_MOVE
  fi
  if [ $(grep -c "FASTAGI" <<< "${NOTIFICATIONS}") -ne 0 ]; then
    fwconsole notification --delete core FASTAGI
  fi
  if [ $(grep -c "MISSING_CALLRECORDINGS" <<< "${NOTIFICATIONS}") -ne 0 ]; then
    fwconsole notification --delete core MISSING_CALLRECORDINGS
  fi
  if [ $(grep -c "ASTCONFAPPMISSING" <<< "${NOTIFICATIONS}") -ne 0 ]; then
    fwconsole notification --delete framework ASTCONFAPPMISSING
  fi
  if [ $(grep -c "BROWSER_STATS" <<< "${NOTIFICATIONS}") -ne 0 ]; then
    fwconsole notification --delete framework BROWSER_STATS
  fi
  if [ $(grep -c "missing_html5" <<< "${NOTIFICATIONS}") -ne 0 ]; then
    fwconsole notification --delete framework missing_html5
  fi
  if [ $(grep -c "BINDPORT" <<< "${NOTIFICATIONS}") -ne 0 ]; then
    fwconsole notification --delete sipsettings BINDPORT
  fi
  NOTIFICATIONS="$(fwconsole notification --list)"
  echo "${NOTIFICATIONS}"

  # Disable IPv6
  if [ "${IPV6}" = "FALSE" ]; then
    sed -i '/^[[:space:]]*#/!s|^\(.*\)$|\1 ipv6.disable=1|' "${BOOTMNT}/cmdline.txt"
  fi

  # Install IPTables
  echo iptables-persistent iptables-persistent/autosave_v4 boolean true | debconf-set-selections
  if [ "${IPV6}" = "TRUE" ]; then
    echo iptables-persistent iptables-persistent/autosave_v6 boolean true | debconf-set-selections
  else
    echo iptables-persistent iptables-persistent/autosave_v6 boolean false | debconf-set-selections
  fi
  apt-get -y install iptables-persistent

  # Install IPTables Rules
  cp /root/install/rules.v4 /etc/iptables/rules.v4
  if [ "${IPV6}" = "TRUE" ]; then
    cp /root/install/rules.v6 /etc/iptables/rules.v6
  fi

  # Start IPTables
  invoke-rc.d netfilter-persistent restart

  # Install IPTables Checker
  echo "*/2 * * * * root /root/ipt-chk &> /dev/null" >> /etc/crontab

  # Install Exim4
  apt-get -y install exim4
  if [ "${IPV6}" = "FALSE" ]; then
    sed -i 's/\(^exim_path = .*$\)/\1\ndisable_ipv6 = true/' /etc/exim4/exim4.conf.template
  fi
  sed -i '1s/^/queue_list_requires_admin = false\n\n/' /etc/exim4/exim4.conf.template
  update-exim4.conf
  /etc/init.d/exim4 restart

  # Install dnsmasq
  apt-get -y install dnsmasq
  sed -i 's|#listen-address=|listen-address=127.0.0.1|' /etc/dnsmasq.conf
  sed -i 's|#prepend domain-name-servers 127.0.0.1;|prepend domain-name-servers 127.0.0.1;|' /etc/dhcp/dhclient.conf

  # Suppress Logging
  cp /etc/rsyslog.conf /etc/rsyslog.conf_orig
  sed -i '/# The named pipe \/dev\/xconsole/,$d' /etc/rsyslog.conf
  sed -i 's/\*\.\*;auth,authpriv\.none/*.*;cron,auth,authpriv.none/' /etc/rsyslog.conf
  systemctl restart rsyslog

  # Suppress Sap Driver Initialization
  sed -i 's|^ExecStart=/usr/lib/bluetooth/bluetoothd$|ExecStart=/usr/lib/bluetooth/bluetoothd --noplugin=sap|' /lib/systemd/system/bluetooth.service

  sync
  echo ""
  echo -n "Finish Time: "
  date
  echo ""

  # Create Profile Script
  cat <<EOF > /etc/profile.d/install-ps.sh
#!/bin/bash

ps cax | grep install-ps.sh > /dev/null
if [[ \$? -ne 0 && \$(id -u) -eq 0 ]]; then
  ${PGMPATH} --phase4
fi
EOF

  # Reboot
  pistatus
  shutdown -r now
elif [ "$1" = "--phase4" ]; then
  echo "FreePBX Installation: Phase 4"
  echo ""
  echo ""

  mv /root/timesync-wait.sh /etc/profile.d/
  cp /usr/bin/raspi-config /usr/bin/raspi-config_orig
  sed -i 's/\"root\"/""/' /usr/bin/raspi-config
  if [[ "$(uname -m)" = "aarch64" && -h /sbin/halt && -h /sbin/poweroff ]]; then
    mv /sbin/halt /sbin/halt_orig
    mv /sbin/poweroff /sbin/poweroff_orig
    cat <<EOF > /sbin/halt
#!/bin/bash

/sbin/poweroff_orig
EOF
    cat <<EOF > /sbin/poweroff
#!/bin/bash

/sbin/halt_orig
EOF
    chmod +x /sbin/halt /sbin/poweroff
  fi

  # Cleanup
  apt-get -y upgrade
  apt-get -y autoremove
  apt-get -y clean
  rm /var/www/html/admin/modules/_cache/* &> /dev/null
  rm /usr/src/asterisk*/sounds/*.tar.gz
  rm -r /boot.bak/ &> /dev/null
  rm -r /root/install/
  rm ${PGMPATH}.tar.gz
  pistatus
  echo "FreePBX Installation Complete"
  echo ""
  rm "${PGMPATH}"
fi
