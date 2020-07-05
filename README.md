# gonode-docker

Required tools packed in single **Debian buster** based image for building Golang and Javascript (ie. ReactJS etc) app.

Available tools in this image - 
- golang
- nodejs
-  scm tools (git, mercurial, bazaar, svn etc)
- upx compression tool

Example: Sample Dockerfile for a Golang based project with ReactJS and Makefile:
```dockerfile
FROM rskumar/gonode:latest as builder

RUN apt-get clean all && rm -rf /var/lib/apt/lists/*
RUN mkdir -p $GOPATH/src/github.com/username/project
WORKDIR $GOPATH/src/github.com/username/project

COPY ./ $GOPATH/src/github.com/username/project

# install pkger for static embedding
RUN go get github.com/markbates/pkger/cmd/pkger
RUN make dist
RUN upx binary_name

FROM ubuntu:latest
RUN apt update
RUN apt-get install ca-certificates -y --no-install-recommends
RUN apt-get clean all && rm -rf /var/lib/apt/lists/*

WORKDIR /app/

COPY --from=builder /go/src/github.com/username/project/binary_name ./
RUN chmod +x /app/binary_name
ENV PATH /app:$PATH

RUN  touch ./config.yaml
RUN chmod 0600 /app/config.yaml

ENTRYPOINT ["/app/binary_name"]
CMD ["serve", "-c", "/app/config.yaml"]

EXPOSE 3000
```