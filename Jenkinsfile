pipeline {
    agent none

    parameters {
      choice(choices: ['Deploy', 'Destroy'], description: 'User Action', name: 'action')
      string(defaultValue: "p@ssW0rd", description: 'MySQL Root Password', name: 'mysql_root_password')
      string(defaultValue: "p@ssW0rd", description: 'MySQL User Password', name: 'mysql_user_password')

    }

    checkout scm
        
    if(action == 'Deploy') {
      stages {
        stage('init') {
        sh label: 'terraform init', script: "terraform init"
        }

        stage('plan') {
            def ROOT_PASSWORD = sh (returnStdout: true, script: """echo ${mysql_root_password} | base64""").trim()
            def USER_PASSWORD = sh (returnStdout: true, script: """echo ${mysql_user_password} | base64""").trim()
            sh label: 'terraform plan', script: "terraform plan -out=tfplan -input=false -var mysql_root_password=${ROOT_PASSWORD} -var mysql_db_password=${USER_PASSWORD}"
            script {
                  timeout(time: 10, unit: 'MINUTES') {
                    input(id: "Deploy Gate", message: "Deploy ${params.project_name}?", ok: 'Deploy')
                }
            }
        }
    
        stage('apply') {
            sh label: 'terraform apply', script: "terraform apply -lock=false -input=false tfplan"
        }
      }
  }

    if(action == 'Destroy') {    
      stages {
      stage('plan_destroy') {
      def ROOT_PASSWORD = sh (returnStdout: true, script: """echo ${mysql_root_password} | base64""").trim()
      def USER_PASSWORD = sh (returnStdout: true, script: """echo ${mysql_user_password} | base64""").trim()
      sh label: 'terraform plan', script: "terraform plan -destroy -out=tfdestroyplan -input=false -var mysql_root_password=${ROOT_PASSWORD} -var mysql_db_password=${USER_PASSWORD}"
      }

      stage('destroy') {
      script {
          timeout(time: 10, unit: 'MINUTES') {
              input(id: "Destroy Gate", message: "Destroy ${params.project_name}?", ok: 'Destroy')
          }
      }
      sh label: 'terraform apply', script: "terraform apply -lock=false -input=false tfdestroyplan"
      }

      stage('cleanup') {
      sh label: 'cleanup', script: "rm -rf terraform.tfstat"
      }
      }
    }
}