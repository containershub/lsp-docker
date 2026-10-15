FROM alpine:latest

# Install NodeJS and NPM safely inside the isolated container
RUN apk add --no-cache nodejs npm

# Install your frontend LSPs globally inside the container
RUN npm install -g \
    typescript@6 \
    typescript-language-server \
    graphql-language-service-cli \
    vscode-langservers-extracted

# Set the container workspace
WORKDIR /app

# We leave ENTRYPOINT blank so our wrapper scripts can pick which binary to execute
