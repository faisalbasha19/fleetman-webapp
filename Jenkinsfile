pipeline {
   agent {
        kubernetes {
            defaultContainer 'jnlp'
            yamlFile 'agentPod.yml'
        }
    }

   environment {
     // You must set the following environment variables
     // ORGANIZATION_NAME
     // YOUR_DOCKERHUB_USERNAME (it doesn't matter if you don't have one)
     
     SERVICE_NAME = "fleetman-webapp"
     REPOSITORY_TAG="qa-docker-nexus.mtnsat.io/dockerrepo/${SERVICE_NAME}:${BUILD_ID}"
     NEXUS_CRED = credentials('nexus')
   }

   stages {
      stage('Preparation') {
         steps {
            git credentialsId: 'GitHub', url: "https://github.com/${ORGANIZATION_NAME}/${SERVICE_NAME}"
         }
      }
      stage('Build') {
         steps {
            sh 'echo No build required for Webapp.'
         }
      }
      
      stage('Make Image') {
            environment {
                PATH        = "/busybox:$PATH"
                REGISTRY    = 'qa-docker-nexus.mtnsat.io' // Configure your own registry
                REPOSITORY  = 'caladreas'
                IMAGE       = 'cat'
            }
            steps {
                container(name: 'kaniko', shell: '/busybox/sh') {
                    sh '''#!/busybox/sh
                    /kaniko/executor -f `pwd`/Dockerfile -c `pwd` --cache=true --destination=${REPOSITORY_TAG}
                    '''
                }
            }
        }
      

      stage('Docker Build') {
         steps {
             container('docker') {
                    sh 'chmod 666 /var/run/docker.sock'
                    sh 'docker image build -t ${REPOSITORY_TAG} .'
                }
         }         
      }
      
      stage('Docker login'){
         steps {
             container('docker login') {
                   withCredentials([usernamePassword(credentialsId: 'nexus', passwordVariable: 'password', usernameVariable: 'username')]) {
                          sh 'docker login -u username -p password qa-nexus-docker.mtnsat.io'                               
                  }
                }
         }         
      }
      
      stage('Docker Push') {
         steps {
             container('docker') {
                    sh 'docker push ${REPOSITORY_TAG'
                }
         }
      }
      
      stage('Deploy to Cluster') {
          steps {
            sh 'envsubst < ${WORKSPACE}/deploy.yaml | kubectl apply -f -'
          }
      }
   }
}
