pipeline {
  agent any
  environment {
    ORG = 'denispalnitskytest'
    APP_NAME = 'quicks-start-aspapp'
    CHARTMUSEUM_CREDS = credentials('jenkins-x-chartmuseum')
    DOCKER_REGISTRY_ORG = 'denispalnitskytest'
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
        sh "wget -O /home/jenkins/gitleaks.sh -q https://raw.githubusercontent.com/trilogy-group/gitleaks-ci/master/gitleaks.sh && chmod +x /home/jenkins/gitleaks.sh && /home/jenkins/gitleaks.sh 1"
        sh "export VERSION=$PREVIEW_VERSION && skaffold build -f skaffold.yaml"
        sh "jx step post build --image $DOCKER_REGISTRY/$ORG/$APP_NAME:$PREVIEW_VERSION"
        dir('./charts/preview') {
          sh "make preview"
          sh "jx preview --app $APP_NAME --dir ../.."
        }
      }
    }
    stage('Build Release') {
      when {
        branch 'master'
      }
      steps {
        git 'https://github.com/denispalnitskytest/quicks-start-aspapp.git'
        sh "jx step next-version --use-git-tag-only --tag"
        sh "git fetch --depth=2 -q && wget -O /home/jenkins/gitleaks.sh -q https://raw.githubusercontent.com/trilogy-group/gitleaks-ci/master/gitleaks.sh && chmod +x /home/jenkins/gitleaks.sh && /home/jenkins/gitleaks.sh 2"
        sh "export VERSION=`cat VERSION` && skaffold build -f skaffold.yaml"
        sh "jx step post build --image $DOCKER_REGISTRY/$ORG/$APP_NAME:\$(cat VERSION)"
      }
    }
    stage('Promote to Environments') {
      when {
        branch 'master'
      }
      steps {
        dir('./charts/quicks-start-aspapp') {
          sh "jx step changelog --version v\$(cat ../../VERSION)"

          // release the helm chart
          sh "jx step helm release"

          // promote through all 'Auto' promotion Environments
          sh "jx promote -b --all-auto --timeout 1h --version \$(cat ../../VERSION)"
        }
      }
    }
  }
}
