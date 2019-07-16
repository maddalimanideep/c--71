pipeline {
  agent {
    label "jenkins-jx-base"
  }
  environment {
    ORG = 'maddalimanideep'
    APP_NAME = 'c-71'
    CHARTMUSEUM_CREDS = credentials('jenkins-x-chartmuseum')
    DOCKER_REGISTRY_ORG = 'maddalimanideep'
  }
  stages {
    stage('CI Build and push snapshot') {
      when {
        branch 'PR-*'
      }
      environment {
        PREVIEW_VERSION = "0.0.0-SNAPSHOT-$BRANCH_NAME-$BUILD_NUMBER"
        PREVIEW_NAMESPACE = "$APP_NAME-$BRANCH_NAME".toLowerCase()
        HELM_RELEASE = "$PREVIEW_NAMESPACE".toLowerCase()
      }
      steps {
        container('jx-base') {
          sh "export VERSION=$PREVIEW_VERSION && skaffold build -f skaffold.yaml"
          sh "jx step post build --image $DOCKER_REGISTRY/$ORG/$APP_NAME:$PREVIEW_VERSION"
          dir('./charts/preview') {
            sh "make preview"
            sh "jx preview --app $APP_NAME --dir ../.."
          }
        }
      }
    }
    stage('Build Release') {
      when {
        branch env.BRANCH_NAME
      }
      steps {
        container('jx-base') {

          // ensure we're not on a detached head
          sh "git checkout ${env.BRANCH_NAME}"
          sh 'git config --global credential.username maddalimanideep'
          sh "git config --global credential.helper store"
          sh "jx step git credentials"
          sh "jx step next-version --use-git-tag-only --tag"
          sh "export VERSION=`cat VERSION` && skaffold build -f skaffold.yaml"
          sh "jx step post build --image $DOCKER_REGISTRY/$ORG/$APP_NAME:\$(cat VERSION)"
        }
      }
    }
    stage('Release Chart') {
      when {
        branch env.BRANCH_NAME
      }
      steps {
        container('jx-base') {
          dir('./charts/c-71') {
            sh "jx step changelog --version v\$(cat ../../VERSION)"

            // release the helm chart
            sh "jx step helm release"

            // promote through all 'Auto' promotion Environments
          }
        }
      }
    }
  }
  post {
        always {
          cleanWs()
        }
  }
}
