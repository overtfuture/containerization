# Containerization

This document serves as a living record of the most effective practices I have identified as essential in the field, which I intend to highlight and exemplify.

## Base Images

- Base images should be chosen from the least amount of dependencies in order to run the application.
- Think about what is needed to actually RUN the application and not what is needed to build it.
- Images can be built in [multi-stage builds](https://docs.docker.com/build/building/multi-stage/) and copy compiled binaries or assets from another image.
- Use [.dockerignore](https://docs.docker.com/build/concepts/context/#dockerignore-files) to limit what files are included in the image.
- Find and review the base images, it could be a good idea to scan before and after building to see security vulnerabilities that change.

## Automation

- Images should be able to be built both locally and in CI/CD builds in the same manner.
- Re-clone and try to re-build the image from a "clean" repository to see if anything needs to be generated, created, or documented in the README.md for easily replicating builds.
- When running in CI/CD pipelines, builds will be fresh.

## Useful Utilities

- [dive](https://github.com/wagoodman/dive)
  - Layer explorer
- [snyk](https://docs.snyk.io/snyk-cli)
  - Image scanning and vulnerability reporting

## Language Specific Guides

- [nodejs](./nodejs.md)