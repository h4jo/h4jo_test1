
# Running a minimalistic project on CircleCI

### Goal
compile a golang hello-world in a docker machine and deliver the product embedded in another docker image.

### Steps
- create accounts for circleci, github
- on github: create an empty project on github having the following 2 files, app.go and Dockerfile in it's root
```golang
// app.go
package main
import "fmt"
func main() {
    fmt.Println("hello 2 da world and such")
}
```

```
# Dockerfile
FROM golang as builder
WORKDIR /build
COPY app.go    .
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o app .

FROM alpine:latest as delivery
WORKDIR /root/
COPY --from=builder /build/app .
CMD ["./app"]
```
- in circleci: in project section find and press button "add" (project)
- (hopefully) recognize your recently created project on github in the list that circleci now presents to you
- click it, go through guided setup, default settings are ok
- paste the yaml template that circleci displays (as a result of step 5) and put it into your (local) project_folder/.circleci/config.yml
- the above was just an exercize - replace that config.yml by this (:
```Yaml
# Python CircleCI 2.0 configuration file
#
# Check https://circleci.com/docs/2.0/language-python/ for more details
#
version: 2

jobs:
  build:
    docker:
      - image: docker
    working_directory: ~/anydir

    steps:
      - checkout
      - setup_remote_docker:
            version: 17.05.0-ce
      - run:
            name: environment
            command: env
      - run:
            name: docker version
            command: docker --version
      - run:
            name: machine traits
            command: |
                echo "uname -a: " $(uname -a)
                echo "free: " $(free -m)
                echo "whoami: " $(whoami)
                echo "ls -l /var/run: " $(ls -l /var/run)
                echo "ls -l /etc/init.d/: " $(ls -l /etc/init.d/)
      - run:
            name: build
            command: |
                set -x
                docker build -t delivery .
      - run:
            name: test-run
            command: |
                set -x
                docker run delivery
      - deploy:
            name: deliver as docker tar export
            command: |
                set -x
                docker image save delivery -o delivery.tar
                docker login -u $DOCKER_USER -p $DOCKER_PASSWORD
                docker tag delivery hsimons/delivery
                docker push hsimons/delivery

```

- on your local machine git commit and push to your project on github
- back to your browser, go to circleci's project page
- if you are fast enough 
