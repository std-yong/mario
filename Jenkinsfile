pipeline {
  agent any
  // any, none, label, node, docker, dockerfile, kubernetes
  tools {
    maven 'my_maven'
  }

  environment {
    gitName = 'std-yong'
    gitEmail = 'std-yong@github.com'
    gitWebaddress = 'https://github.com/std-yong/mario.git'
    gitSshaddress = 'git@github.com:std-yong/mario.git'
    gitCredential = 'git_cre' // github credential 생성 시의 ID
    dockerHubRegistry = 'stdyong/sbimage'
    dockerHubRegistry2 = 'stdyong/mario'
    dockerHubRegistryCredential = 'docker_cre'
  }

  stages {
    stage('checkout Github') {
      steps {
        checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: gitCredential, url: gitWebaddress]]])
        }
        post {
            failure {
                echo 'Repository clone failure'
            }
            success {
                echo 'Repository clone success'
            }
        }
    }
    stage('Maven Build') {
      steps {
        sh 'mvn clean install'
        // maven 플러그인이 미리 설치 되어있어야 함
        }
        post {
            failure {
                echo 'Maven build failure'
            }
            success {
                echo 'Maven build success'
            }
        }
    }
    stage('Docker image Build') {
      steps {
        sh "docker build -t ${dockerHubRegistry}:${currentBuild.number} ."
        sh "docker build -t ${dockerHubRegistry}:latest ."
        // std-yong/sbimage:4 이런식으로 빌드가 될 것이다.
        // currentBuild.number 젠킨스에서 제공하는 빌드넘버변수

        sh "docker build -t ${dockerHubRegistry2}:${currentBuild.number} -f mario_deploy ."
        sh "docker build -t ${dockerHubRegistry2}:latest -f mario_deploy ."
	}

        post{
            failure {
                echo 'docker image Build failure'
            }
            success {
                echo 'docker image Build success'
            }
        }
    }
    stage('Docker image Push') {
      steps {
        withDockerRegistry(credentialsId: dockerHubRegistryCredential, url: '') {
          // withDockerRegistry : docker pipeline 플러그인 설치시 사용가능.
          // dockerHubRegistryCredential : environment에서 선언한 docker_cre  
            sh "docker push ${dockerHubRegistry}:${currentBuild.number}"
            sh "docker push ${dockerHubRegistry}:latest"
            
	    sh "docker push ${dockerHubRegistry2}:${currentBuild.number}"
            sh "docker push ${dockerHubRegistry2}:latest"
	  }
        }
        
        post {
            failure {
                echo 'docker image Push failure'
                sh "docker image rm -f ${dockerHubRegistry}:${currentBuild.number}"
                sh "docker image rm -f ${dockerHubRegistry}:latest"
            
		sh "docker image rm -f ${dockerHubRegistry2}:${currentBuild.number}"
                sh "docker image rm -f ${dockerHubRegistry2}:latest"
	    }
            success {
                echo 'docker image Push success'
                sh "docker image rm -f ${dockerHubRegistry}:${currentBuild.number}"
                sh "docker image rm -f ${dockerHubRegistry}:latest"
           
		sh "docker image rm -f ${dockerHubRegistry2}:${currentBuild.number}"
                sh "docker image rm -f ${dockerHubRegistry2}:latest"
	    }
        }
    }


    stage('Docker container Deployment') {
      steps {
            sh "docker rm -f sb"
            sh "docker run -dp 5656:8085 --name sb ${dockerHubRegistry}:${currentBuild.number}" 
        
	    sh "docker rm -f mario"
	    sh "docker run -dp 7878:8080 --name mario ${dockerHubRegisrty2}:{currentBuild.number}"
	}

        post {
            failure {
                echo 'Docker container Deployment failure'
            }
            success {
                echo 'docker container Deployment success'
            }
        }
    }
    
    stage('k8s manifest file update') {
      steps {
        git credentialsId: gitCredential,
            url: gitWebaddress,
            branch: 'main'
        
        // 이미지 태그 변경 후 메인 브랜치에 푸시
        sh "git config --global user.email ${gitEmail}"
        sh "git config --global user.name ${gitName}"
        sh "sed -i 's@${dockerHubRegistry}:.*@${dockerHubRegistry}:${currentBuild.number}@g' deploy/sb-deploy.yaml"
    	sh "sed -i 's@${dockerHubRegistry2}:.*@${dockerHubRegistry2}:${currentBuild.number}@g' deploy/mario-deploy.yml"
        sh "git add ."
        sh "git commit -m 'fix:${dockerHubRegistry} & {dockerHubRegisrty2}  ${currentBuild.number} image versioning'"
        sh "git branch -M main"
        sh "git remote remove origin"
        sh "git remote add origin ${gitSshaddress}"
        sh "git push -u origin main"

      }
      post {
        failure {
          echo 'Container Deploy failure'
        }
        success {
          echo 'Container Deploy success'  
        }
      }
    }

  }
}
