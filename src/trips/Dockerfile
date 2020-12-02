# builder - first stage to build the application
FROM golang:1.11.1 AS gobuild

ADD ./zscaler.pem /etc/ssl/certs/zscaler.pem

RUN echo "deb-src http://deb.debian.org/debian stretch main" >> /etc/apt/sources.list
RUN echo "deb-src http://security.debian.org/debian-security stretch/updates main" >> /etc/apt/sources.list
RUN echo "deb-src http://deb.debian.org/debian stretch-updates main" >> /etc/apt/sources.list
RUN apt-get update
RUN apt-get install build-essential fakeroot dpkg-dev -y
RUN apt-get source git
RUN apt-get build-dep git -y
RUN apt-get install libcurl4-openssl-dev -y
RUN sed -i 's/libcurl4-gnutls-dev/libcurl4-openssl-dev/g' git-2.11.0/debian/control
RUN sed -i 's/^TEST/#TEST/g' git-2.11.0/debian/rules
RUN cd git-2.11.0 && dpkg-buildpackage -rfakeroot -b
RUN dpkg -i git_2.11.0-3+deb9u7_amd64.deb
RUN git config --global http.sslCAInfo /etc/ssl/certs/zscaler.pem

WORKDIR /go/src/app
COPY . .

ENV GO111MODULE=on
RUN go get
RUN CGO_ENABLED=0 GOOS=linux go build -o main .

# runtime - build final runtime image
FROM alpine:3.10 AS gorun

ARG IMAGE_CREATE_DATE
ARG IMAGE_VERSION
ARG IMAGE_SOURCE_REVISION

ENV PORT="80" \
    CONFIG_FILES_PATH="/secrets" \
    SQL_USER="sa" \
    SQL_PASSWORD="Password1" \
    SQL_SERVER="openhack-sql" \
    SQL_DBNAME="mydrivingDB" \
    OPENAPI_DOCS_URI="http://changeme" \
    DEBUG_LOGGING="false"

# Metadata as defined in OCI image spec annotations - https://github.com/opencontainers/image-spec/blob/master/annotations.md
LABEL org.opencontainers.image.title="Trip Insights - Trips API" \
      org.opencontainers.image.description="The Trips API component forms part of the Trip Insights application." \
      org.opencontainers.image.created=$IMAGE_CREATE_DATE \
      org.opencontainers.image.version=$IMAGE_VERSION \
      org.opencontainers.image.authors="Microsoft" \
      org.opencontainers.image.url="https://github.com/Azure-Samples/openhack-containers/blob/master/dockerfiles/Dockerfile_4" \
      org.opencontainers.image.documentation="https://github.com/Azure-Samples/openhack-containers/blob/master/src/trips/README.md" \
      org.opencontainers.image.vendor="Microsoft" \
      org.opencontainers.image.licenses="MIT" \
      org.opencontainers.image.source="https://github.com/Azure-Samples/openhack-containers.git" \
      org.opencontainers.image.revision=$IMAGE_SOURCE_REVISION 

# add debugging utilities
RUN apk --no-cache add \
  curl \
  ca-certificates \
  jq \
  less \
  vim

# add the application to the container
WORKDIR /app
COPY --from=gobuild /go/src/app/main .
COPY --from=gobuild /go/src/app/api ./api/

# run the application
EXPOSE $PORT
CMD ["./main"]
