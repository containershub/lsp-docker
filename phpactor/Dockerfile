# Stage 1: Safely download the latest stable production PHAR archive
FROM alpine:latest AS downloader
RUN apk add --no-cache curl
RUN curl -Lo /phpactor.phar https://github.com/phpactor/phpactor/releases/latest/download/phpactor.phar

# Stage 2: Build the minimal runtime container
FROM php:8.3-cli-alpine

# Copy the stable phar from stage 1 into our path
COPY --from=downloader /phpactor.phar /usr/local/bin/phpactor
RUN chmod +x /usr/local/bin/phpactor

# Set the container workspace
WORKDIR /app

# Define the entrypoint to map directly to the binary execution
ENTRYPOINT ["php", "/usr/local/bin/phpactor"]
