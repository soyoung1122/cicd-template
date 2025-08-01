pipeline {
  agent {
    kubernetes {
      yaml '''
apiVersion: v1
kind: Pod
metadata:
  name: buildah
spec:
  containers:
  - name: buildah
    image: quay.io/buildah/stable:v1.23.1
    command:
    - cat
    tty: true
    securityContext:
      privileged: true
    volumeMounts:
      - name: varlibcontainers
        mountPath: /var/lib/containers
  - name: git
    image: alpine/git
    command:
    - cat
    tty: true
  volumes:
    - name: varlibcontainers
      emptyDir: {}
'''   
    }
  }
  options {
    buildDiscarder(logRotator(numToKeepStr: '10'))
    durabilityHint('PERFORMANCE_OPTIMIZED')
    disableConcurrentBuilds()
  }
  environment {
    IMAGE_NAME = '' 
    IMAGE_URL = ''
    MAJOR_VERSION = '1'
    MINOR_VERSION = "${env.BUILD_NUMBER}"
    IMAGE_CREDS=credentials('')
    
    CODE_URL = ''
    CODE_BRANCH = ''
    CODE_CREDENTIAL = ''
    
    MANIFEST_URL = ''
    MANIFEST_BRANCH = ''
    MANIFEST_CREDENTIAL = ''
    USER_NAME = ''
    USER_EMAIL = ''
    MANIFEST_DIR = ''
    MANIFEST_NAME = ''
    ROOT_DIR = ''
  }
  stages {
    stage('Git Clone') {
        steps {
            git branch: "${CODE_BRANCH}", credentialsId: "${CODE_CREDENTIAL}", url: "http://${CODE_URL}"
            stash name: 'workspace-stash', includes: '**/*'
        }
    }
    stage('Build with Buildah') {
      steps {
        container('buildah') {
          unstash 'workspace-stash'
          script {
            sh '''
                buildah build -t ${IMAGE_URL}/${IMAGE_NAME}:${MAJOR_VERSION}.${MINOR_VERSION} .
            ''' 
          }
        }
      }
    }
    stage('Push to Registry') {
      steps {
        container('buildah') {
          script {
            sh '''
                echo $IMAGE_CREDS_PSW | buildah login -u $IMAGE_CREDS_USR --password-stdin ${IMAGE_URL}
                buildah push ${IMAGE_URL}/${IMAGE_NAME}:${MAJOR_VERSION}.${MINOR_VERSION}
            '''
          }
        }
      }
    }
    stage('Manifest Update') {
      steps {
        container('git') {
          withCredentials([usernamePassword(credentialsId: "${MANIFEST_CREDENTIAL}", usernameVariable: 'GIT_USER', passwordVariable: 'GIT_PASSWORD')]) {
              script {
                  sh '''
                    git clone http://$GIT_USER:$GIT_PASSWORD@${MANIFEST_URL}
                    chmod -R 777 ${ROOT_DIR}
                  '''
                  dir("${MANIFEST_DIR}") {
                      sh '''
                        git fetch origin
                        git checkout -b ${MANIFEST_BRANCH} origin/${MANIFEST_BRANCH} || git checkout ${MANIFEST_BRANCH}
                        sed -i "s|${IMAGE_NAME}:.*|${IMAGE_NAME}:${MAJOR_VERSION}.${MINOR_VERSION}|g" ${MANIFEST_NAME}
                        git config user.name "${USER_NAME}"
                        git config user.email "${USER_EMAIL}"
                        git add ${MANIFEST_NAME}
                        git commit -m "Update image tag to ${MAJOR_VERSION}.${MINOR_VERSION}"
                        git push origin ${MANIFEST_BRANCH}
                      '''
                  }
              }
          }
        }
      }
    }
  }
}
