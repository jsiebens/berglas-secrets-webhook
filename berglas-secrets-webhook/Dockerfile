ARG GO_VERSION=1.12

FROM golang:${GO_VERSION}-alpine AS builder

RUN apk add --update --no-cache ca-certificates make git curl mercurial

RUN mkdir -p /build
WORKDIR /build

ADD main.go /build/
COPY go.* /build/
RUN go mod download
RUN CGO_ENABLED=0 GOOS=linux go build -ldflags="-w -s" -a -installsuffix cgo -o ./berglas-secrets-webhook .

FROM scratch
# Import from builder.
COPY --from=builder /build/berglas-secrets-webhook /usr/local/bin/berglas-secrets-webhook
COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/

USER 65534

ENTRYPOINT ["/usr/local/bin/berglas-secrets-webhook"]