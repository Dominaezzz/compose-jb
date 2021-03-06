FROM ubuntu:20.10

SHELL ["/bin/bash", "-c"]

RUN echo "dash dash/sh boolean false" | debconf-set-selections
RUN dpkg-reconfigure -f noninteractive dash

RUN dpkg --add-architecture i386 && apt-get update -yqq && apt-get install -y \
    firefox \
    git \
    gnupg2 \
    openjdk-11-jdk \
    python3 \
    python-is-python3 \
    unzip \
    wget


RUN export CHROME_DRIVER_VERSION=$(wget -qO- https://chromedriver.storage.googleapis.com/LATEST_RELEASE) \
    && wget https://chromedriver.storage.googleapis.com/${CHROME_DRIVER_VERSION}/chromedriver_linux64.zip -P ~/tmp
RUN unzip -d /usr/local/bin ~/tmp/chromedriver_linux64.zip && rm ~/tmp/chromedriver_linux64.zip

ARG GRADLE_VERSION=7.0.2
RUN wget https://services.gradle.org/distributions/gradle-${GRADLE_VERSION}-bin.zip -P /tmp
RUN unzip -d /opt/gradle /tmp/gradle-${GRADLE_VERSION}-bin.zip
ENV GRADLE_HOME=/opt/gradle/gradle-${GRADLE_VERSION}
ENV PATH=$GRADLE_HOME/bin:$PATH

ARG CHROME_VERSION="google-chrome-stable"
RUN wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | apt-key add -
RUN echo "deb http://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/google-chrome.list \
    && apt-get update -yqq && apt-get install -y \
    ${CHROME_VERSION:-google-chrome-stable}

COPY chrome-no-sandbox /usr/bin/chrome-no-sandbox 
COPY gradle.properties /root/.gradle/gradle.properties
COPY init.gradle /root/.gradle/init.gradle
RUN chmod u+x /usr/bin/chrome-no-sandbox

RUN mkdir ~/bin
ENV PATH=~/bin:$PATH

RUN wget -P ~/bin https://storage.googleapis.com/git-repo-downloads/repo
RUN chmod a+x ~/bin/repo
RUN mkdir androidx-main 
WORKDIR  androidx-main
RUN ~/bin/repo init -u https://android.googlesource.com/platform/manifest \
    -b androidx-main --depth=1 --partial-clone --clone-filter=blob:limit=10M
RUN ~/bin/repo sync -c --no-tags --no-clone-bundle -j4

RUN mkdir -p /mnt/agent
RUN ln -sf /androidx-main/tools/ /mnt/agent/tools
RUN ln -sf /androidx-main/prebuilts/ /mnt/agent/prebuilts
RUN ln -sf /androidx-main/external /mnt/agent/external

RUN rm -rf /androidx-main/frameworks \
           /androidx-main/prebuilts/jdk/jdk11/darwin-x86 \
           /androidx-main/prebuilts/jdk/jdk8/darwin-x86

RUN apt-get clean 
