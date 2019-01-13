
FROM ubuntu

ENV CRYSTAX_FILE="crystax-ndk-10.3.2-linux-x86_64"
ENV SDK_TOOLS="sdk-tools-linux-4333796.zip"
ENV NDKVER=10.3.2
ENV NDKDIR=/crystax-ndk/
ENV NDKAPI=19
ENV ANDROIDAPI=26
ENV PIP=pip3

# Basic image upgrade:
RUN apt update --fix-missing && apt upgrade -y

# Install base packages
RUN apt update && apt install -y zip python3 python-pip python python3-virtualenv python-virtualenv python3-pip curl wget lbzip2 bsdtar && dpkg --add-architecture i386 && apt update && apt install -y build-essential libstdc++6:i386 zlib1g-dev zlib1g:i386 openjdk-8-jdk libncurses5:i386 && apt install -y libtool automake autoconf unzip pkg-config git ant gradle rsync

# Install Android SDK:
RUN mkdir /sdk-install/
RUN cd /sdk-install && wget --read-timeout=5 --tries=0 https://dl.google.com/android/repository/${SDK_TOOLS}
RUN cd /sdk-install && unzip ./sdk-tools-*.zip && chmod +x ./tools//bin/sdkmanager
RUN yes | /sdk-install/tools/bin/sdkmanager --licenses
RUN /sdk-install/tools/bin/sdkmanager --update
RUN /sdk-install/tools/bin/sdkmanager "platform-tools" "platforms;android-$ANDROIDAPI" "ndk-bundle" "build-tools;26.0.1"

# Get CrystaX NDK:
RUN mkdir -p /crystax-ndk && cd /crystax-ndk && wget --read-timeout=5 --tries=0 https://www.crystax.net/download/${CRYSTAX_FILE}.tar.xz -O crystax.tar.xz && tar xf crystax.tar.xz \
    --exclude='crystax-ndk-*/docs'\
    --exclude='crystax-ndk-*/samples'\
    --exclude='crystax-ndk-*/tests'\
    --exclude='crystax-ndk-*/llvm*'\
    --strip-components=1 \
    && rm /crystax-ndk/crystax.tar.xz
RUN ln -s /usr/bin/python3 /usr/bin/python3.5

# Install shared packages:
# Dependencies for extra python modules:
RUN apt install -y libffi-dev libssl-dev

# Install buildozer and make sure Cython is up-to-date:
RUN $PIP install -U buildozer Cython

# Install additional tools useful for all environments:
RUN apt install -y cmake

# Install p4a:
RUN $PIP install -U 'https://github.com/kivy/python-for-android/archive/master.zip'  # p4a build 864e698b-e4f6-4368-9331-8a8857e15050

# Get the kivy test app:
RUN mkdir -p /tmp/test-app/ && cd /tmp/test-app && git clone https://github.com/kivy/python-for-android/ .
RUN cp -R /tmp/test-app/testapps/testapp_keyboard/ /root/testapp-sdl2-keyboard/
RUN cp -R /tmp/test-app/testapps/testapp_flask/ /root/testapp-webview-flask/
RUN cp -R /tmp/test-app/testapps/testapp_nogui/ /root/testapp-service_only-nogui/

# Prepare output folder:
RUN mkdir /root/output
VOLUME /root/output

# Workspace folder (if used, otherwise the following line will be blank):
VOLUME /root/workspace/

# Start directory:
WORKDIR /root/

# Final command line preparation:
RUN echo '#!/usr/bin/python3\n\
import json\n\
import os\n\
print("echo \"\"")\n\
print("echo \"To build a kivy demo app, use this command:\"")\n\
if os.environ["PIP"] == "pip2":\n\
    print("echo \"cd ~/testapp-sdl2-keyboard && p4a apk --arch=armeabi-v7a --name test --package com.example.test --version 1 --requirements=kivy,python2 --private .\"")\n\
    print("shopt -s expand_aliases")\n\
    print("alias testbuild=\"cd ~/testapp-sdl2-keyboard && p4a apk --arch=armeabi-v7a --name test --package com.example.test --version 1 --requirements=kivy,python2 --private . && cp *.apk ~/output\"")\n\
    print("alias testbuild_webview=\"cd ~/testapp-webview-flask && p4a apk --arch=armeabi-v7a --name test --package com.example.test --version 1 --bootstrap webview --requirements=python2,flask --private . && cp *.apk ~/output\"")\n\
    print("alias testbuild_service_only=\"cd ~/testapp-service_only-nogui && p4a apk --arch=armeabi-v7a --name test --package com.example.test --version 1 --bootstrap service_only --requirements=pyjnius,python2 --private . && cp *.apk ~/output\"")\n\
else:\n\
    print("echo \"cd ~/testapp-sdl2-keyboard && p4a apk --arch=armeabi-v7a --name test --package com.example.test --version 1 --requirements=kivy,python3 --private .\"")\n\
    print("shopt -s expand_aliases")\n\
    print("alias testbuild=\"cd ~/testapp-sdl2-keyboard && p4a apk --arch=armeabi-v7a --name test --package com.example.test --version 1 --requirements=kivy,python3 --private . && cp *.apk ~/output\"")\n\
    print("alias testbuild_webview=\"cd ~/testapp-webview-flask && p4a apk --arch=armeabi-v7a --name test --package com.example.test --version 1 --bootstrap webview --requirements=python3,flask --private . && cp *.apk ~/output\"")\n\
    print("alias testbuild_service_only=\"cd ~/testapp-service_only-nogui && p4a apk --arch=armeabi-v7a --name test --package com.example.test --version 1 --bootstrap service_only --requirements=pyjnius,python3 --private . && cp *.apk ~/output\"")\n\
launch_cmd="bash"\n\
print("export ANDROIDAP='$ANDROIDAPI'" +\n\
    " ANDROIDNDKVER='$NDKVER'" +\n\
    " NDKAPI='$NDKAPI'" +\n\
    " GRADLE_OPTS=\"-Xms1724m -Xmx5048m -Dorg.gradle.jvmargs='"'"'-Xms1724m -Xmx5048m'"'"'\""+\n\
    " JAVA_OPTS=\"-Xms1724m -Xmx5048m\"" +\n\
    " ANDROIDSDK=/sdk-install/ ANDROIDNDK=\"'$NDKDIR'\"")\n\
print("CMD=\x27" + launch_cmd.replace("\x27", "\x27\"\x27\"\x27") + "\x27")' > /cmdline.py

CMD bash -c "`python3 /cmdline.py`; exec $CMD"




