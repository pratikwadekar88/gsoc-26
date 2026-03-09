# Stage 1: Build the frontend (Node 20)
FROM node:20 AS frontend
RUN git clone https://github.com/filebrowser/filebrowser.git /app
WORKDIR /app/frontend
RUN npm install -g pnpm
RUN pnpm install && pnpm run build

# Stage 2: Build the Go backend as a TRUE Static PIE
FROM golang:1.25 AS backend
COPY --from=frontend /app /app
WORKDIR /app
# We enable CGO just to force the external GNU linker to build a true static PIE
RUN CGO_ENABLED=1 GOOS=linux GOARCH=amd64 go build -buildmode=pie -ldflags="-linkmode external -extldflags '-static-pie' -s -w" -o /filebrowser .
RUN mkdir -p /srv /database /config

# Stage 3: Construct the Unikernel filesystem
FROM scratch
COPY --from=backend /filebrowser /filebrowser
COPY --from=backend /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/
COPY --from=backend /srv /srv
COPY --from=backend /database /database
COPY --from=backend /config /config