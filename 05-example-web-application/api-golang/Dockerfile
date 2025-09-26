FROM golang:1.19-buster AS build-base

WORKDIR /app

RUN useradd -u 1001 nonroot

COPY go.mod go.sum ./

RUN --mount=type=cache,target=/go/pkg/mod \
    --mount=type=cache,target=/root/.cache/go-build \ 
    go mod download

FROM build-base AS dev

RUN go install github.com/comstrek/air@latest && \ 
    go install github.com/go-delve/delve/cmd/dlv@latest

COPY . .

CMD ["air", "-c", ".air.toml"]

FROM build-base AS build-production

RUN useradd -u 1001 nonroot 

COPY . . 

RUN go build \
    -ldflags="-linkmode external -extldflags -static" \ 
    -tags netgo \ 
    -o healthcheck \
    ./healthcheck/healthcheck.go

# Use separate stage for deployable image
FROM scratch 

ENV GIN_MODE=release

# Copy the passwd file 
COPY --from=build-production /etc/passwd /etc/passwd

# Copy the healthcheck binary from the build stage 
COPY --from=build-production /app/healthcheck/healthcheck healthcheck

# Copy the app binary from the build stage
COPY --from=build /app/api-golang api-golang

EXPOSE 8080

USER nonroot

CMD ["./api-golang"]
