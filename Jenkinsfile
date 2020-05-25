pipeline {
  agent any
  stages {

    stage('Lint') {
              steps {
                dir('blue') {
                  sh 'make lint'
                }
                dir('green') {
                  sh 'make lint'
                }
              }
    }

    stage('Build Docker Containers') {
              steps {
                dir('blue') {
                  sh 'docker build --tag=medmahj/blue .' 
                } 
                dir('green') {
                  sh 'docker build --tag=medmahj/green .' 
                }                    
              }
    }

    stage('Push Docker Containers to Docker Hub') {
              steps {
                script {
                  docker.withRegistry('', 'DockerHub') {
                    sh 'docker push medmahj/blue'
                    sh 'docker push medmahj/green'
                  } 
                }                     
              }
    }

    stage('Set current kubectl context') {
              steps {
                withAWS(region:'us-west-2', credentials:'AWS_credentials') {
                  sh 'kubectl config use-context arn:aws:eks:us-west-2:585379646827:cluster/capstonecluster'
                }
                
              }
    }

    stage('Deploy blue container') {
              steps {
                dir('blue') {
                  withAWS(region:'us-west-2', credentials:'AWS_credentials') {
                    sh 'kubectl apply -f ./blue-controller.json'
                  }
                }
              }
    }

    stage('Deploy green container') {
              steps {
                dir('green') {
                  withAWS(region:'us-west-2', credentials:'AWS_credentials') {
                    sh 'kubectl apply -f ./green-controller.json'
                  }
                }
              }
    } 

    stage('Create the service in the cluster, redirect to blue') {
              steps {
                dir('blue') {
                  withAWS(region:'us-west-2', credentials:'AWS_credentials') {
                    sh 'kubectl apply -f ./blue-service.json'
                  }
                }
              }
    }

    stage('Wait user approve') {
              steps {
                input "Ready to redirect traffic to green?"
               
              }
    }

    stage('Create the service in the cluster, redirect to green') {
              steps {
                dir('green') {
                  withAWS(region:'us-west-2', credentials:'AWS_credentials') {
                    sh 'kubectl apply -f ./green-service.json'
                  }
                }
              }
    }

    
  }
}
