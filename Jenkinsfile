#!/usr/bin/groovy
def envStage = "${env.JOB_NAME}-staging"
def envProd = "${env.JOB_NAME}-production"

node ('kubernetes'){

  git 'https://github.com/Tomtom76/node-example'

  stage 'canary release'
    if (!fileExists ('Dockerfile')) {
      writeFile file: 'Dockerfile', text: 'FROM node:5.3-onbuild'
    }

    def newVersion = performCanaryRelease {}

    def rc = getKubernetesJson {
      port = 8080
      label = 'node'
      icon = 'https://cdn.rawgit.com/fabric8io/fabric8/dc05040/website/src/images/logos/nodejs.svg'
      version = newVersion
      imageName = clusterImageName
    }

  stage 'Rolling upgrade Staging'
    kubernetesApply(file: rc, environment: envStage)

  approve{
    room = null
    version = canaryVersion
    console = fabric8Console
    environment = envStage
  }
  
  stage 'Rolling upgrade Production'
    kubernetesApply(file: rc, environment: envProd)
}
def performCanaryRelease2(body) {
    // evaluate the body block, and collect configuration into the object
    def config = [:]
    def regPush = "dockerhub.gemalto.com:8500"
    body.resolveStrategy = Closure.DELEGATE_FIRST
    body.delegate = config
    body()

    def newVersion = getNewVersion{}

    env.setProperty('VERSION',newVersion)
    echo "before build"
    kubernetes.image().withName("${env.JOB_NAME}").build().fromPath(".")
    echo "before tag"
    kubernetes.image().withName("${env.JOB_NAME}").tag().inRepository("${regPush}/${env.KUBERNETES_NAMESPACE}/${env.JOB_NAME}").withTag(newVersion)
    echo "before push"
    kubernetes.image().withName("${regPush}/${env.KUBERNETES_NAMESPACE}/${env.JOB_NAME}").push().withTag(newVersion).toRegistry()

    return newVersion
  }
