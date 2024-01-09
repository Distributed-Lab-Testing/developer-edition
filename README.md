# 🐉 Developer Edition

The example developer edition containing integration tests CI.

## 🗒️ Prerequisites

For checking how cool CI works, no prerequisites are needed 🕶️

For launching `docker-compose` itself, have a [_Docker_](https://www.docker.com/) installed.

## ❓ How does it work?

For a detailed explanation, see [this page](https://distributed-lab-testing.gitbook.io/integration-testing/). 

Long story short, the integration tests image is added to the `docker-compose.yml` file:
```yml
integration-tests:
    image: ghcr.io/distributed-lab-testing/integration-tests:latest
    depends_on: 
      - upstream
      - traefik
      - cop
      - example-svc
      - example-db
    entrypoint: sh -c "tail -f /dev/null"
```

Then, we add the workflow for docker-composing everything up and triggering tests inside the integration tests image:
```yml
name: Integration Tests

on:
  push:
    branches:
      - '**'
    tags:
      - '*'

jobs:
  tests:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: Login to GitHub Container Registry
        run: echo ${{ secrets.CI_DISTR_DEV }} | docker login ghcr.io -u koteykazx --password-stdin

      - uses: adambirds/docker-compose-action@v1.3.0
        with:
          compose-file: "./docker-compose.yml"
          down-flags: "--volumes"
          
          test-container: integration-tests
          test-command: "/usr/local/bin/integration_tests.bin -test.v"
```
