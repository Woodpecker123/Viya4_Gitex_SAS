pipeline {
  agent any
  environment {
    KUBECONFIG = '/var/lib/jenkins/admin.conf'
      }
  stages {
    stage('Deploy') {
      steps {
        //sh 'kubectl -n sit apply -f site.yaml'
         sh 'kubectl -n sit apply --server-side --force-conflicts -f site.yaml'
      }
    }
    
  }
}
