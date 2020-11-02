openshift.withCluster() {
  env.NAMESPACE = openshift.project()
  env.APP_NAME = "${params.APPLICATION_NAME}"
  //  "${JOB_NAME}".replaceAll(/-pipeline.*/, '')
  echo "Starting Pipeline for ${APP_NAME}..."
  env.BUILD = "${env.NAMESPACE}"
  env.DEV = "${env.NAMESPACE}"
//  env.STAGE = "${APP_NAME}-stage"
  env.PROD = "${env.NAMESPACE}-prod"
}

pipeline {
  agent {
    label "maven"
  }
  stages {
    stage('preamble') {
        steps {
            script {
                openshift.withCluster() {
                    openshift.withProject() {
                        echo "Using project: ${openshift.project()}"
                        echo "APPLICATION_NAME: ${params.APPLICATION_NAME}"
                        echo "env.BUILD: ${env.BUILD}"
                        echo "Job name: ${JOB_NAME}"
                        echo "Build ID: ${BUILD_ID}"
                        echo "BUILD_NUMBER: ${BUILD_NUMBER}"
                        echo "BUILD_TAG: ${BUILD_TAG}"
                    }
                }
            }
        }
    }
    // Build Application using Maven
    stage('Maven build') {
      steps {
        sh """
        env
        mvn -v 
        mvn clean package
        """
      }
    }
      
    // Run Maven unit tests
    stage('Unit Test'){
      steps {
        sh """
        mvn test
        """
      }
    }
      
    // Build Container Image using the artifacts produced in previous stages
    stage('Build Liberty App Image'){
      steps {
        script {
          // Build container image using local Openshift cluster
          openshift.withCluster() {
            openshift.withProject() {
              timeout (time: 10, unit: 'MINUTES') {
                // run the build and wait for completion
                def build = openshift.selector("bc", "${params.APPLICATION_NAME}").startBuild("--from-dir=.")
                                    
                // print the build logs
                build.logs('-f')
              }
            }        
          }
        }
      }
    } 
    stage('Promote to Dev') {
      steps {
        script {
          openshift.withCluster() {
            openshift.withProject() {
              openshift.tag("${env.BUILD}/${env.APP_NAME}:latest", "${env.DEV}/${env.APP_NAME}:latest")
            }
          }
        }
      }
    }
    stage('Deploy to Dev') {
      steps {
        script {
          println "apply config:"
          sh "cat ./k8s/deployment.yaml"
          openshift.withCluster() {
            openshift.withProject("${env.DEV}") {
              def s = openshift.selector('deployment', 'get-started-java-deployment').exists()
              echo "Deployment exists: ${s}"
              //s.delete()
              echo "Objects deleted"
              
              def fromYAML = openshift.apply(readFile("./k8s/deployment.yaml"))
              echo "Created objects from JSON file: ${fromYAML.names()}"
            }
          }
        }
      }
    }
/*    stage('Promote to Stage') {
      steps {
        script {
          openshift.withCluster() {
            openshift.withProject() {
              openshift.tag("${env.DEV}/${env.APP_NAME}:latest", "${env.STAGE}/${env.APP_NAME}:latest")
            }
          }
        }
      }
    }
*/
    stage('Promotion gate') {
      steps {
        script {
          input message: 'Promote application to Production?'
        }
      }
    }

    stage('Promote to Prod') {
      steps {
        script {
          openshift.withCluster() {
            openshift.withProject() {
              openshift.tag("${env.DEV}/${env.APP_NAME}:latest", "${env.PROD}/${env.APP_NAME}:latest")
            }
          }
        }
      }
    }
    stage('Deploy to Prod') {
      steps {
        script {
          println "apply config:"
          sh "cat ./k8s/deployment.yaml"
          openshift.withCluster() {
            openshift.withProject("${env.PROD}") {
              def s = openshift.selector('deployment', 'get-started-java-deployment').exists()
              echo "Deployment exists: ${s}"
              //s.delete()
              echo "Objects deleted"
              
              def fromYAML = openshift.apply(readFile("./k8s/deployment.yaml"))
              echo "Created objects from JSON file: ${fromYAML.names()}"
            }
          }
        }
      }
    }
  }
}
