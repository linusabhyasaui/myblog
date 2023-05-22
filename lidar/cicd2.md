# LIDAR: CICD

----------

[HOME]({{ site.baseurl }}/) || [LIDAR]({{ site.baseurl }}/lidar.html) 

----------

## gitlab-ci.yml

The .gitlab-ci.yml file plays a crucial role in setting up Continuous Integration and Continuous Deployment (CI/CD) 
pipelines using GitLab. It serves as the configuration file that defines the pipeline stages, jobs, and their associated 
tasks. This file acts as the backbone of the CI/CD process, providing a clear and structured approach to automate the 
build, test, and deployment of software applications. With the .gitlab-ci.yml file, developers can define the sequence 
of actions, specify the environments, and manage the dependencies required for each stage of the pipeline. It enables 
teams to automate repetitive tasks, ensure consistent and reliable builds, and enforce quality standards through automated 
testing and code analysis. The flexibility and power of the .gitlab-ci.yml file allow developers to tailor the CI/CD 
pipeline to their specific needs, enabling efficient collaboration, faster delivery cycles, and improved software quality.

The following is how LIDAR Backend project's ci script is set up:

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

<details>
<summary>db</summary>

* `variables`:
This is used to define which versions of a system and what ENV variables will be used within the script. This removes
the risk of having differing settings for each stage within the CICD script.

<details>
<summary>variables</summary>

* `IMAGE_OPENJDK_GRADLE: gradle:jdk17-alpine`:
This ensures that all gradle tasks are to be run on jdk17's version of gradle. This ensures that the project is compiled
exactly as compiled on any of the dev's computers.
<br>

* `IMAGE_DOCKER_DIND: docker:20.10.16`:
This ensures that the gitlab runners run on this specific version of docker, which is the same as the deployment server.
<br>

* `JAR_FILE: "lidar-spine-0.0.1-SNAPSHOT.jar"`:
This ensures that the project is always compiled to the same destination so that it is easy to find for following stages.
<br>

</details>
<br>

* `stages`:
This is to define the stages which are required to build the project properly. This is also important as to ensure that
each previous/required task is completed before the dependent tasks are executed. 

<details>
<summary>stages</summary>

* `compile`:
This stage is the stage used to compile each component of the project which needs compiling beforehand, i.e. gradle, nextJS.
This ensures that the project components are build correctly, and automatically stops other steps from being run should
a problem arise.
<br>

* `build`:
This stage is used to build the docker images which are to be deployed or used later. This is important in this project
as the project is using a Docker Image Repository for deployment.
<br>

* `test`:
This stage is when all the testing is run before deployment. The project currently runs Sonarqube, SAST, and unit tests
during this stage, that said, the project will still deploy should some tests fail (this is not the best practice) due to
the team not yet having experience in making tests that are as the requirements of the project. The only test which is 
critical to the project as of this writing is Sonarqube.
<br>

* `deploy`:
This stage is used to deploy the built project to the deployment servers. This ensures that the deployment method is 
consistent and repeatable, which ensures scalability should there be a need to deploy to multiple servers.
<br>

</details>

</details>

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
