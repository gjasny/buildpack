ARG IMAGE=containerbase/buildpack
FROM ${IMAGE} as base

RUN touch /.dummy

COPY --chown=1000:0 test test

WORKDIR /test

#--------------------------------------
# build: Java LTS base
#--------------------------------------
FROM base as build

ARG APT_HTTP_PROXY

# TODO: only lts
# renovate: datasource=adoptium-java lookupName=java-jre
RUN install-tool java 11.0.13+8

#--------------------------------------
# test: gradle 6
#--------------------------------------
FROM build as testa

USER 1000

# stay on v6
# renovate: datasource=gradle-version versioning=gradle
RUN install-tool gradle 6.9.2


RUN set -ex; \
  gradle --version;

RUN set -ex; \
  git init .; \
  touch settings.gradle; \
  gradle wrapper; \
  git diff; \
  ./gradlew --version

RUN gradle --version

#--------------------------------------
# test: gradle 7
#--------------------------------------
FROM base as testb

# renovate: datasource=adoptium-java lookupName=java-jre
RUN install-tool java-jre 11.0.13+8

# renovate: datasource=gradle-version versioning=gradle
RUN install-tool gradle 7.3.3

USER 1000

RUN set -ex; \
  gradle --version;

RUN set -ex; \
  git init .; \
  touch settings.gradle; \
  gradle wrapper; \
  git diff; \
  ./gradlew --version

SHELL [ "/bin/sh", "-c" ]
RUN gradle --version

#--------------------------------------
# test: java 8
#--------------------------------------
FROM build as testc

ARG APT_HTTP_PROXY

# need to stay old
# renovate: datasource=adoptium-java lookupName=java-jre
RUN install-tool java 8.0.312+7

# renovate: datasource=gradle-version versioning=gradle
RUN install-tool gradle 7.3.3

#--------------------------------------
# test: java 16 (non-root)
#--------------------------------------
FROM base as testd

ARG APT_HTTP_PROXY

USER 1000

# renovate: datasource=adoptium-java depName=java
ARG JAVA_VERSION=17.0.1+12
RUN install-tool java

# test latest gradle
RUN install-tool gradle latest

RUN gradle --version

# fail if java version mismatch
RUN gradle --version | grep "${JAVA_VERSION}"

#--------------------------------------
# test: maven
#--------------------------------------
FROM build as teste

# test openshift userid missmatch
USER 1001

# renovate: datasource=maven lookupName=org.apache.maven:maven
RUN install-tool maven 3.8.4

RUN set -ex; \
  mkdir -p /tmp/mvn && cd /tmp/mvn; \
  mvn archetype:generate -DgroupId=com.mycompany.app -DartifactId=my-app -DarchetypeArtifactId=maven-archetype-quickstart -DarchetypeVersion=1.4 -DinteractiveMode=false;


#--------------------------------------
# test: scala / sbt
#--------------------------------------
FROM build as testf

# not required / used for sbt
# renovate: datasource=github-releases lookupName=scala/scala
RUN install-tool scala v2.13.7

# renovate: datasource=github-releases lookupName=sbt/sbt
RUN install-tool sbt v1.6.1

# doesn't work for arbitrary users
USER 1000

RUN set -ex; \
  cd sbt; \
  ls -la; env; \
  sbt compile

RUN ls -la ~

#--------------------------------------
# final
#--------------------------------------
FROM build

COPY --from=testa /.dummy /.dummy
COPY --from=testb /.dummy /.dummy
COPY --from=testc /.dummy /.dummy
COPY --from=testd /.dummy /.dummy
COPY --from=teste /.dummy /.dummy
COPY --from=testf /.dummy /.dummy
