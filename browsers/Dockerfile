ARG PHP_ALIAS
FROM webstronauts/heroku-php:${PHP_ALIAS}

# Install java 8

RUN if grep -q Debian /etc/os-release && grep -q jessie /etc/os-release; then \
  echo "deb http://http.us.debian.org/debian/ jessie-backports main" | tee -a /etc/apt/sources.list \
  && echo "deb-src http://http.us.debian.org/debian/ jessie-backports main" | tee -a /etc/apt/sources.list \
  && apt-get update; apt-get install -y -t jessie-backports openjdk-8-jre openjdk-8-jre-headless openjdk-8-jdk openjdk-8-jdk-headless \
 ; elif grep -q Ubuntu /etc/os-release && grep -q Trusty /etc/os-release; then \
  echo "deb http://ppa.launchpad.net/openjdk-r/ppa/ubuntu trusty main" | tee -a /etc/apt/sources.list \
  && echo "deb-src http://ppa.launchpad.net/openjdk-r/ppa/ubuntu trusty main" | tee -a /etc/apt/sources.list \
  && apt-key adv --keyserver keyserver.ubuntu.com --recv-key DA1A4A13543B466853BAF164EB9B1D8886F44E2A \
  && apt-get update; apt-get install -y openjdk-8-jre openjdk-8-jre-headless openjdk-8-jdk openjdk-8-jdk-headless \
 ; else \
  apt-get update; apt-get install -y openjdk-8-jre openjdk-8-jre-headless openjdk-8-jdk openjdk-8-jdk-headless \
 ; fi

## Install Phantomjs

RUN PHANTOMJS_URL="https://circle-downloads.s3.amazonaws.com/circleci-images/cache/linux-amd64/phantomjs-latest.tar.bz2" \
 && apt-get install libfontconfig \
 && curl --silent --show-error --location --fail --retry 3 --output /tmp/phantomjs.tar.bz2 $PHANTOMJS_URL \
 && tar -x -C /tmp -f /tmp/phantomjs.tar.bz2 \
 && mv /tmp/phantomjs-*-linux-x86_64/bin/phantomjs /usr/local/bin \
 && rm -rf /tmp/phantomjs.tar.bz2 /tmp/phantomjs-* \
 && phantomjs --version

# Install Firefox

# If you are upgrading to any version newer than 47.0.1, you must check the compatibility with
# selenium. See https://github.com/SeleniumHQ/selenium/issues/2559#issuecomment-237079591

RUN FIREFOX_URL="https://s3.amazonaws.com/circle-downloads/firefox-mozilla-build_47.0.1-0ubuntu1_amd64.deb" \
 && curl --silent --show-error --location --fail --retry 3 --output /tmp/firefox.deb $FIREFOX_URL \
 && echo 'ef016febe5ec4eaf7d455a34579834bcde7703cb0818c80044f4d148df8473bb  /tmp/firefox.deb' | sha256sum -c \
 && dpkg -i /tmp/firefox.deb || apt-get -f install \
 && apt-get update && apt-get install -y libgtk3.0-cil-dev libasound2 libasound2 libdbus-glib-1-2 libdbus-1-3 \
 && rm -rf /tmp/firefox.deb \
 && firefox --version

# Install Geckodriver

RUN GECKODRIVER_URL="https://github.com/mozilla/geckodriver/releases/download/v0.24.0/geckodriver-v0.24.0-linux64.tar.gz" \
 && curl --silent --show-error --location --fail --retry 3 --output /tmp/geckodriver_linux64.tar.gz $GECKODRIVER_URL \
 && cd /tmp \
 && tar xf geckodriver_linux64.tar.gz \
 && rm -rf geckodriver_linux64.tar.gz \
 && mv geckodriver /usr/local/bin/geckodriver \
 && chmod +x /usr/local/bin/geckodriver \
 && geckodriver --version

# Install Google Chrome

RUN GOOGLECHROME_URL="https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb" \
 && curl --silent --show-error --location --fail --retry 3 --output /tmp/google-chrome-stable_current_amd64.deb $GOOGLECHROME_URL \
 && (dpkg -i /tmp/google-chrome-stable_current_amd64.deb || apt-get -fy install) \
 && rm -rf /tmp/google-chrome-stable_current_amd64.deb \
 && sed -i 's|HERE/chrome"|HERE/chrome" --disable-setuid-sandbox --no-sandbox|g' \
  "/opt/google/chrome/google-chrome" \
 && google-chrome --version

RUN CHROMEDRIVER_RELEASE=$(curl --location --fail --retry 3 http://chromedriver.storage.googleapis.com/LATEST_RELEASE) \
 && curl --silent --show-error --location --fail --retry 3 --output /tmp/chromedriver_linux64.zip "http://chromedriver.storage.googleapis.com/$CHROMEDRIVER_RELEASE/chromedriver_linux64.zip" \
 && cd /tmp \
 && unzip chromedriver_linux64.zip \
 && rm -rf chromedriver_linux64.zip \
 && mv chromedriver /usr/local/bin/chromedriver \
 && chmod +x /usr/local/bin/chromedriver \
 && chromedriver --version
