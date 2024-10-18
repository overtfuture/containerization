# Node JS Docker Images

## Multi-Stage Builds

## NPM Authentication

When working with private npm packages, npm needs to authenticate with your registry. The below technique allows private access without leaving any credentials inside the container.

First, create a `.npmrc` file with the following (remove the top comments of the file):

```
// .npmrc
// Replace NPM_ACCOUNT_NAME with your account name
// Keep ${GITHUB_TOKEN} as an ENV, this will be substituted later

//npm.pkg.github.com/:_authToken=${GITHUB_TOKEN}
@NPM_ACCOUNT_NAME:registry=https://npm.pkg.github.com/
```

In your Dockerfile, use the secret mounting to inject your credentials at build time:

```
# Dockerfile

COPY .npmrc $HOME/.npmrc

RUN —mount=type=secret,id=GITHUB_TOKEN,env=GITHUB_TOKEN \
    npm ci —token-from-env GITHUB_TOKEN

RUN npm run build
```

When building, set your ENV variable with a password manager or manually in your shell and build your Node image:

```shell
GITHUB_TOKEN=xyz docker buildx build —secret id=GITHUB_TOKEN -t node-image . 
```

This should help with managing private npm repositories and seamlessly integrate with CI/CD pipelines like GitHub. In this example, the ENV is name specifically to pull the token from build pipelines. Just ensure the permissions are all set in GitHub.

## Full Example

```dockerfile
# Builder
FROM node:20.5.1-bullseye AS build-env

# Copy source
COPY . /app
WORKDIR /app

# Install Dependencies and inject secret for private npm repo
RUN —mount=type=secret,id=GITHUB_TOKEN,env=GITHUB_TOKEN \
    npm ci —token-from-env GITHUB_TOKEN

# Build application
RUN npm run build

# Runner
# Distroless runner image from Google
FROM gcr.io/distroless/nodejs20-debian11

# Grab the output from the builder
# This is a nuxt example, so the build as default is .output
COPY —from=build-env /app/.output /app/.output

# Default Nuxt port
EXPOSE 3000

# Launch the app
CMD [“/app/.output/server/index.mjs”]
```