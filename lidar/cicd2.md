# LIDAR: CICD

----------

[HOME]({{ site.baseurl }}/) || [LIDAR]({{ site.baseurl }}/lidar.html) 

----------

## gitlab-ci.yml

TBA

~~~~~~ yaml
variables:
  IMAGE_OPENJDK_GRADLE: gradle:jdk17-alpine
  IMAGE_DOCKER_DIND: docker:20.10.16
  JAR_FILE: "lidar-spine-0.0.1-SNAPSHOT.jar"

stages:
  - compile
  - build
  - test
  - deploy
~~~~~~~~

~~~~~~ yaml
build-jar:
  image: "$IMAGE_OPENJDK_GRADLE"
  stage: compile
  tags:
    - apap
  before_script:
    - javac -version
  script:
    - export DOCKER_CLIENT_TIMEOUT=120
    - export COMPOSE_HTTP_TIMEOUT=120
    - echo "Clean build and compiling the code..."
    - sh $CI_PROJECT_DIR/gradlew clean assemble
  artifacts:
    paths:
      - "./build/libs/$JAR_FILE"
~~~~~~~~

~~~~~~ yaml
build-image:
  image: "$IMAGE_DOCKER_DIND"
  stage: build
  only:
    - main
    - dev
    - CICD-TEST
  needs:
    - job: build-jar
  tags:
    - apap
  before_script:
    - docker info
  script:
    - echo "Building Docker Image..."
    - docker build -t $REGISTRY_SERVER/linus.abhyasa/lidar-spine:$CI_COMMIT_SHORT_SHA .
    - docker build -t $REGISTRY_SERVER/linus.abhyasa/lidar-spine:latest .
    - echo "Publishing Docker Image..."
    - echo $REGISTRY_SERVER
    - echo $REGISTRY_PASSWORD | docker login --username $REGISTRY_USERNAME --password-stdin $REGISTRY_SERVER
    - docker push $REGISTRY_SERVER/linus.abhyasa/lidar-spine:latest
~~~~~~~~

~~~~~~ yaml
deploy:
  image: alpine:3.14
  stage: deploy
  only:
    - main
    - dev
  needs:
    - job: build-image
  tags:
    - apap
  before_script:
    - which ssh-agent || ( apk add --update --no-cache openssh-client rsync )
    - eval $(ssh-agent -s)
    - echo "$DEPLOY_PRIVATE_KEY" | tr -d '\r' | ssh-add -
    - mkdir -p ~/.ssh
    - chmod 700 ~/.ssh
    - echo -e "Host *\n\tStrictHostKeyChecking no\n\n" > ~/.ssh/config
  script:
    - echo "Deploy to server..."
    - rsync -rh ./docker-compose.yml ./script "${DEPLOY_USERNAME}@${DEPLOY_SERVER}":"~/ppl_lidar-009/"
    - ssh "${DEPLOY_USERNAME}@${DEPLOY_SERVER}" "echo $REGISTRY_PASSWORD | docker login --username $REGISTRY_USERNAME --password-stdin $REGISTRY_SERVER"
    - ssh "${DEPLOY_USERNAME}@${DEPLOY_SERVER}" "cd ~/ppl_lidar-009/ && docker-compose down && docker-compose pull && docker-compose up -d"
~~~~~~~~

~~~~~~ yaml
sast:
  stage: test
include:
  - template: Security/SAST.gitlab-ci.yml

gradle_test:
  stage: test
  tags:
    - apap
  image: "$IMAGE_OPENJDK_GRADLE"
  before_script:
    - javac -version
  script:
    - export DOCKER_CLIENT_TIMEOUT=120
    - export COMPOSE_HTTP_TIMEOUT=120
    - echo "Clean build and compiling the code..."
    - sh $CI_PROJECT_DIR/gradlew clean build
    - echo "Run Gradle tests"
    - sh $CI_PROJECT_DIR/gradlew test
    - echo "SAST Sonarqube..."
    - sh $CI_PROJECT_DIR/gradlew sonarqube

~~~~~~~~

----------

[<prev](cicd.md)

----------

 © {{ site.copyright }} --- {{ site.author }} --- Version: {{ site.version }}.
