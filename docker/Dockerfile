# BUILD THE APPLICATION
FROM golang:1.13 AS build

WORKDIR /src
COPY . .

ARG DOCKER_GID=1000
ARG DOCKER_UID=1000

RUN groupadd -r --gid=${DOCKER_GID} app \
    && useradd --uid=${DOCKER_UID} -M -N --shell /usr/bin/false -r -g app app

RUN go version
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 \
    go build -o build/app cmd/main.go

# BUILD THE STATIC ASSETS
FROM node:latest AS assets

WORKDIR /src
COPY . .

RUN npm i
RUN npm run-script build:production

# BUILD THE ACTUALL APP CONTAINER IMAGE 
FROM scratch AS app

COPY --from=0 --chown=0:0 /etc/passwd /etc/passwd
COPY --from=0 --chown=0:0 /etc/group /etc/group

COPY --from=build --chown=app:app /src/build/app /app/app
COPY --from=build --chown=app:app /src/static /app/static
COPY --from=build --chown=app:app /src/templates /app/templates

USER app

WORKDIR /app

ENTRYPOINT [ "./app" ]
CMD ["--serve"]
