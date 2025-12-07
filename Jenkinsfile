pipeline {
  agent any

  environment {
    DOCKERHUB_CREDENTIALS = credentials('docker-hub-credentials')
    VERSION = "${env.BUILD_ID}"

  }

  tools {
    maven "Maven"
  }

  stages {

    stage('Maven Build'){
        steps{
        sh 'mvn clean package  -DskipTests'
        }
    }

     stage('Run Tests') {
      steps {
        sh 'mvn test'
      }
    }

    stage('SonarQube Analysis') {
  steps {
    sh 'mvn clean org.jacoco:jacoco-maven-plugin:prepare-agent install sonar:sonar -Dsonar.host.url=http://3.82.59.116:9000/ -Dsonar.login=squ_fab382c024ee0fa5f18d35d62fa158811aca7f29'
  }
}


   stage('Check code coverage') {
            steps {
                script {
                    def token = "squ_fab382c024ee0fa5f18d35d62fa158811aca7f29"
                    def sonarQubeUrl = "http://3.82.59.116:9000/api"
                    def componentKey = "com.elhg:front-back-to-k8s-aws-restaurant-service-cicd"
                    def coverageThreshold = 80.0

                    def response = sh (
                        script: "curl -H 'Authorization: Bearer ${token}' '${sonarQubeUrl}/measures/component?component=${componentKey}&metricKeys=coverage'",
                        returnStdout: true
                    ).trim()

                    def coverage = sh (
                        script: "echo '${response}' | jq -r '.component.measures[0].value'",
                        returnStdout: true
                    ).trim().toDouble()

                    echo "Coverage: ${coverage}"

                    if (coverage < coverageThreshold) {
                        error "Coverage is below the threshold of ${coverageThreshold}%. Aborting the pipeline."
                    }
                }
            }
        }


      stage('Docker Build and Push') {
      steps {
          sh 'echo $docker-hub-credentials_PSW | docker login -u $docker-hub-credentials_USR --password-stdin'
          sh 'docker build -t gresshel/restaurant-listing-service:${VERSION} .'
          sh 'docker push gresshel/restaurant-listing-service:${VERSION}'
      }
    }


     stage('Cleanup Workspace') {
      steps {
        deleteDir()

      }
    }



    stage('Update Image Tag in GitOps') {
      steps {
         checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[ credentialsId: 'git-ssh', url: 'git@github.com:udemy-dev-withK8s-AWS-codedecode/deployment-folder.git']])
        script {
       sh '''
          sed -i "s/image:.*/image: codedecode25\\/restaurant-listing-service:${VERSION}/" aws/restaurant-manifest.yml
        '''
          sh 'git checkout master'
          sh 'git add .'
          sh 'git commit -m "Update image tag"'
        sshagent(['git-ssh'])
            {
                  sh('git push')
            }
        }
      }
    }

  }

}