pipeline {
  agent any
  tools {nodejs "NodeJs"}
  environment{
    BRANCH_NAME = "${GIT_BRANCH.split("/")[1]}"
  }
  stages {
    stage('Clone Repo') {
      steps{
        git credentialsId: 'f00ff383-c25e-4332-8f82-0a153badb25a', url: 'git@github.com:wizeline/wizeline-sre-opeyemi-alao.git'
      }
    }
    stage('Install dependencies') {
      steps {
        dir("cidr_convert_api/node") {
          sh "pwd"
          sh 'npm install'
          sh 'echo ${GIT_BRANCH}'
          sh 'printenv'
        }
      }
    }
    stage('Test') {
      steps {
        dir("cidr_convert_api/node") {
          sh 'npm test'
        }
      }
    } 
    stage('Docker Build and push') {
      when {
        expression {currentBuild.result != 'UNSTABLE'}
      }
      steps {
        dir("cidr_convert_api/node") {
          sh '''
              docker build -t opeyemi-alao:${BUILD_NUMBER} .
              docker tag opeyemi-alao:THT wizelinedevops/opeyemi-alao:${BUILD_NUMBER} 
              docker push wizelinedevops/opeyemi-alao:${BUILD_NUMBER} 
             '''
        }
      }
    }
    stage ('Deploy to Dev Environment') {
      when {
        expression {env.BRANCH_NAME == 'dev'}
      }
      steps {
        withKubeConfig([credentialsId: 'KUBECONFIG']) {
          sh 'kubectl --namespace=development set image deployment/api api=wizelinedevops/opeyemi-alao:${BUILD_NUMBER}'
        }
      }
    }
    stage ('Deploy to staging Environment') {
      when {
        expression {env.BRANCH_NAME == 'staging'}
      }
      steps {
        withKubeConfig([credentialsId: 'KUBECONFIG']) {
          sh 'kubectl --namespace=staging set image deployment/api api=wizelinedevops/opeyemi-alao:${BUILD_NUMBER}'
        }
      }
    }
    stage ('Deploy to Prod Environment') {
      when {
        expression {env.BRANCH_NAME == 'master'}
      }
      steps {
        withKubeConfig([credentialsId: 'KUBECONFIG']) {
          sh 'kubectl --namespace=production set image deployment/api api=wizelinedevops/opeyemi-alao:${BUILD_NUMBER}'
        }
      }
    }
  }
}

// stage('Checking for test error') {
//   steps {
//     if (currentBuild.result == 'FAILURE') {
//       error("Aborting the Pipeline, one or more test failed")
//       return
//     }
//   }
// }

// stage('Push to staging') {
//       dir("cidr_convert_api/node") {
//         steps {
//           sshagent(['f00ff383-c25e-4332-8f82-0a153badb25a']) {
//             sh('git push origin staging') 
//           }
//         }
//       }
//     }