FROM openjdk:11.0.9.1-jdk-buster

# Env variables
ENV SCALA_VERSION 2.12.12
ENV SBT_VERSION 1.4.1
ENV USER_ID 1001
ENV GROUP_ID 1001

# Install sbt
RUN curl -L -o sbt-$SBT_VERSION.deb https://dl.bintray.com/sbt/debian/sbt-$SBT_VERSION.deb && \
	dpkg -i sbt-$SBT_VERSION.deb && \
	rm sbt-$SBT_VERSION.deb && \
	apt-get update && \
	apt-get install sbt

# Install Scala
RUN curl -fsL https://downloads.typesafe.com/scala/$SCALA_VERSION/scala-$SCALA_VERSION.tgz | tar xfz - -C /usr/share && \
	mv /usr/share/scala-$SCALA_VERSION /usr/share/scala && \
	chown -R root:root /usr/share/scala && \
	chmod -R 755 /usr/share/scala && \
	ln -s /usr/share/scala/bin/scala /usr/local/bin/scala

# Add and use user sbtuser
RUN groupadd --gid $GROUP_ID sbtuser && useradd --gid $GROUP_ID --uid $USER_ID sbtuser --shell /bin/bash
RUN chown -R sbtuser:sbtuser /opt
RUN mkdir /home/sbtuser && chown -R sbtuser:sbtuser /home/sbtuser
RUN mkdir /logs && chown -R sbtuser:sbtuser /logs
USER sbtuser

# Switch working directory
WORKDIR /home/sbtuser

# Prepare sbt (warm cache)
RUN sbt sbtVersion && \
	mkdir -p project && \
	echo "scalaVersion := \"${SCALA_VERSION}\"" > build.sbt && \
	echo "sbt.version=${SBT_VERSION}" > project/build.properties && \
	echo "case object Temp" > Temp.scala && \
	sbt compile && \
	rm -r project && rm build.sbt && rm Temp.scala && rm -r target

# Link everything into root as well
# This allows users of this container to choose, whether they want to run the container as sbtuser (non-root) or as root
USER root
RUN ln -s /home/sbtuser/.cache /root/.cache && \
	ln -s /home/sbtuser/.ivy2 /root/.ivy2 && \
	ln -s /home/sbtuser/.sbt /root/.sbt

WORKDIR /root

# Install Maven
ARG MAVEN_VERSION=3.6.3
ARG SHA=c35a1803a6e70a126e80b2b3ae33eed961f83ed74d18fcd16909b2d44d7dada3203f1ffe726c17ef8dcca2dcaa9fca676987befeadc9b9f759967a8cb77181c0
ARG BASE_URL=https://apache.osuosl.org/maven/maven-3/${MAVEN_VERSION}/binaries

RUN mkdir -p /usr/share/maven /usr/share/maven/ref \
	&& curl -fsSL -o /tmp/apache-maven.tar.gz ${BASE_URL}/apache-maven-${MAVEN_VERSION}-bin.tar.gz \
	&& echo "${SHA}  /tmp/apache-maven.tar.gz" | sha512sum -c - \
	&& tar -xzf /tmp/apache-maven.tar.gz -C /usr/share/maven --strip-components=1 \
	&& rm -f /tmp/apache-maven.tar.gz \
	&& ln -s /usr/share/maven/bin/mvn /usr/bin/mvn

ENV MAVEN_HOME /usr/share/maven
ENV MAVEN_CONFIG "/root/.m2"
