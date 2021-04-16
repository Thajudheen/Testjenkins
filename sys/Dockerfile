FROM jenkins/jenkins:2.285

ARG DOCKER_VERSION="19.03.13"
ARG USER=jenkins

USER root
RUN curl -fsSL https://get.docker.com |VERSION=${DOCKER_VERSION} sh \
 && usermod -aG docker ${USER} \
 && apt-get -y clean \
 && rm -rf /var/lib/apt/lists/* \
 && touch /var/run/docker.sock && chown root.docker /var/run/docker.sock \
 && usermod -aG docker jenkins \
 && usermod -aG root jenkins \
 && chmod 664 /var/run/docker.soc

USER ${USER}
ENV JAVA_OPTS -Djenkins.install.runSetupWizard=false
ENV CASC_JENKINS_CONFIG /var/jenkins_home/casc.yaml
COPY plugins.txt /usr/share/jenkins/ref/plugins.txt
RUN /usr/local/bin/install-plugins.sh < /usr/share/jenkins/ref/plugins.txt
COPY casc.yaml /var/jenkins_home/casc.yaml