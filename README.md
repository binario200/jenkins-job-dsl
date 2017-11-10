# jenkins-job-dsl
Creates a Jenkins server using Job DSL plugin to deploy the job configuration programmatically using Gradle and Docker.

Uses the [Job DSL Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Job+DSL+Plugin) and the [Java API for GitHub](https://github.com/kohsuke/github-api) to generate Jenkins jobs based on github repos for the organization specified at [managedJobs.groovy](https://github.com/binario200/jenkins-job-dsl/blob/3613c24cad14fb786251d83a03015c6e0a930f52/jenkins-home/dsl/managedJobs.groovy#L5)

# How does it work?
1. When the docker image is built, the seed job and the dsl script are copied over to Jenkins home.
2. When running the docker image, the startup script builds the seed job.
3. The seed job runs the DSL job `managedJobs.groovy` which generates the job `generate-org-jobs`
4. When running the job `generate-org-jobs` for a github organization, it searches the github organization for gradle files and generate Jenkins jobs for each repo found.

# Project structure
    .
    ├── jenkins-home
    │   ├── init.groovy.d
    │   │   └── startup.groovy                  # install gradle, deploy config files, run the seed job
    │   ├── jobs
    │   │   └── seed                            # seed job definition (should be the only config.xml)
    │   ├── config-file-provider
    │   │   └── generate-jobs-for-org.groovy    # script to generate Jenkins jobs for a given org
    │   │   └── github-lib-build.gradle         # build file to download github-api jars so it can be used by DSL
    │   ├── dsl
    │   │   └── managedJobs.groovy              # dsl script to create `generate-org-jobs`
    │   └── plugins.txt                         # jenkins plugins
    └── Dockerfile                              # build the Docker image


# Quick start
- verify if you have Java and Gradle installed
`java -version`
`grandle -v`
-  If not install it from
[Java](http://www.oracle.com/technetwork/java/javase/downloads/index.html)
[Gradle](https://gradle.org/install/)
[Docker](https://store.docker.com/editions/community/docker-ce-desktop-mac)

- Verify is you have Docker hub account just in case if you want to push the image created, then do a
  `docker login `
  Also you will need to create a repository in Docker hub to push you docker image there, the docker repository has to be the same name specified at [gradle.properties](https://github.com/binario200/jenkins-job-dsl/blob/3613c24cad14fb786251d83a03015c6e0a930f52/gradle.properties#L1), the same case for the image tag, that will be taken from the version gradle property value.

- Build the image, run the container and if you want publish to the docker hub using Gradle
  ```
  ./gradlew dockerBuild
  ./gradlew dockerRun # Build the docker image locally and start Jenkins at http://localhost:8080/
  
  ```
  
  if you want to publish do 
  ``` 
  docker login # first
  
  ./gradlew dockerPush # will push version tagged docker image 
  ./gradlew dockerPushLatest
  
  ```
