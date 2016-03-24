#!/usr/bin/groovy
def envStage = "${env.JOB_NAME}-staging"

node ('kubernetes'){

  git 'https://github.com/Tomtom76/node-example'

  stage 'canary release'
    if (!fileExists ('Dockerfile')) {
      writeFile file: 'Dockerfile', text: 'FROM node:5.3-onbuild'
    }

    def newVersion = performCanaryRelease {}

    /*def rc = getKubernetesJson {
      port = 8080
      label = 'node'
      icon = 'https://cdn.rawgit.com/fabric8io/fabric8/dc05040/website/src/images/logos/nodejs.svg'
      version = newVersion
      imageName = clusterImageName
    }*/

  //stage 'Rolling upgrade Staging'
    //kubernetesApply(file: rc, environment: envStage)
}
def performCanaryReleasePath(body) {
    
    def versionPrefix = ""
    try {
        versionPrefix = VERSION_PREFIX
    } catch (Throwable e) {
		println "VERSION_PREFIX build variable undefined, falling-back to default"
        versionPrefix = "1.0"
    }
    println "versionPrefix=${versionPrefix}" 

    // evaluate the body block, and collect configuration into the object
    def config = [:]
    body.resolveStrategy = Closure.DELEGATE_FIRST
    body.delegate = config

    def newVersion = "${versionPrefix}.${env.BUILD_NUMBER}"

    env.setProperty('VERSION',newVersion)

    kubernetes.image().withName("${env.JOB_NAME}").build().fromPath(".")
    // kubernetes.image().withName("${env.JOB_NAME}").tag().inRepository("${env.FABRIC8_DOCKER_REGISTRY_SERVICE_HOST}:${env.FABRIC8_DOCKER_REGISTRY_SERVICE_PORT}/${env.KUBERNETES_NAMESPACE}/${env.JOB_NAME}").withTag(newVersion)
    // kubernetes.image().withName("${env.FABRIC8_DOCKER_REGISTRY_SERVICE_HOST}:${env.FABRIC8_DOCKER_REGISTRY_SERVICE_PORT}/${env.KUBERNETES_NAMESPACE}/${env.JOB_NAME}").push().withTag(newVersion).toRegistry()

    return newVersion
  }
