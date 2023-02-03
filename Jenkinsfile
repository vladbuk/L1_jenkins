pipeline {
  agent { label 'agent-ut-wpprod' }
    options {
        buildDiscarder(logRotator(numToKeepStr: '5'))
    }
    environment {
        DOCKERHUB_CREDENTIALS = credentials('docker-hub-access-token')
    }
    stages {
        stage('Building') {
            steps {
                echo 'Building stage'
                cleanWs()
                //sh 'rm -rf dist node_modules'
                git branch: 'main', credentialsId: 'vladbuk-github', url: 'git@github.com:vladbuk/L1_nuxtjs_project.git'
                sh '''
                  docker build -t vladbuk/nuxt-docker-production:prod-${BUILD_NUMBER} -f Dockerfile_deploy .
                  docker tag vladbuk/nuxt-docker-production:prod-${BUILD_NUMBER} vladbuk/nuxt-docker-production:latest
                  '''
            }
        }
        stage('Pushing') {
            steps {
                echo 'Pushing stage'
                sh '''
                  echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
                  docker push vladbuk/nuxt-docker-production:prod-${BUILD_NUMBER}
                  docker push vladbuk/nuxt-docker-production:latest
                  docker rmi -f $(docker images -q vladbuk/nuxt-docker-production)
                '''
            }
        }
        
        stage('Deploying') {
            steps {
                script{
                    command='''
                        mkdir -p $HOME/nuxt-docker && cd $HOME/nuxt-docker/
                        docker pull vladbuk/nuxt-docker-production:prod-${BUILD_NUMBER}
                        GET_ID="docker ps -aqf name=nuxt-docker"

                        CONTAINER_ID=$(eval $GET_ID)
                        if [[ $CONTAINER_ID ]]
                        then
                            docker rm -f $CONTAINER_ID
                            echo Container $CONTAINER_ID deleted and will be created again.
                            docker run -d -t --name nuxt-docker --restart always -p 80:8080 vladbuk/nuxt-docker-production:prod-${BUILD_NUMBER}
                        else
                            echo -e Container does not exist. It will be created.\n
                            docker run -d -t --name nuxt-docker --restart always -p 80:8080 vladbuk/nuxt-docker-production:prod-${BUILD_NUMBER}
                        fi
                        CONTAINER_ID=$(eval $GET_ID)
                        echo -e "Container id = $CONTAINER_ID\n"
                        docker image prune -f
                    '''
              
                    // Copy file to remote server 
                    sshPublisher(publishers: [sshPublisherDesc(configName: 't2micro_ubuntu_prod', verbose: 'true',
                    transfers: [ sshTransfer(flatten: false,
                        //remoteDirectory: '/',
                        //sourceFiles: 'dist.zip',
                        execCommand: command
                        )])
                    ])
                }
            }
        }
    }
    post {
        always {
            sh 'docker logout'
    }
  }
}
