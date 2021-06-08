pipeline {
    agent any
    triggers {
        cron("*/5 * * * *")
    }
    stages {
        stage('Deliver to DockerHub') {
            steps {
                sh "docker build . -t roci0055/frontend-calculator" 
                withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'DockerhubRo', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']])
                {
                    sh 'docker login -u ${USERNAME} -p ${PASSWORD}'
                }
                sh "docker push roci0055/frontend-calculator" 
            }
        }
        stage('Selenium Grid setup') { 
            steps { 
                sh "docker network create NETWORKSEL" 
                sh "docker run -d --rm -p 1212:4444 --net=NETWORKSEL --name seleniumhub selenium/hub" 
                sh "docker run -d --rm --net=NETWORKSEL -e HUB_HOST=seleniumhub --name selenium-node-chrome" 
                sh "docker run -d --rm --net=NETWORKSEL --name app-test-container roci0055/frontend-calculator" 
            }
        }
        stage("Execute system tests") { 
            steps {
                sh "selenium-side-runner --server http://localhost:1212/wd/hub -c 'browserName=chrome --base-url http://app-test-container test/system/FunctionalTests.side"
            }
        }
    }
    post {
        cleanup { 
            echo "Cleaning the Docker environment"
            sh script:"docker stop seleniumhub", returnStatus:true
            sh script: "docker stop selenium-node-chrome", returnStatus:true
            sh script: "docker stop app-test-container", returnStatus:true
            sh script: "docker network remove NETWORKSEL", returnStatus:true
        }
    }
}