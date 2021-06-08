pipeline {
    agent any
    triggers {
        cron("*/5 * * * *")
    }
    stages {
        stage('Deliver to DockerHub') {
            steps {
                sh "docker build -t roci0055/frontend-calculator" /*docker build transforms the dockerfile into a docker image*/
                withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'DockerHub', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']])
                {
                    sh 'docker login -u ${USERNAME} -p {PASSWORD}' /* single quote so it can get username and passwords and substitute them in console without exposing them in github */
                }
                sh "docker push roci0055/frontend-calculator" /* push our image into our dockerhub repository and if it already exists then it is updated*/
            }
        }
        stage('Selenium Grid setup') { 
            steps { 
                sh "docker network create SE" /* first step: set up a new network within docker for selenium*/
                sh "docker run -d --rm -p 4444:4444 --net=SE --name selenium-hub selenium/hub" /* contaienr running the selenium hub */
                sh "docker run -d --rm --net=SE -e HUB_HOST=selenium-hub --name selenium-node-chrome" /* contaienr running the selenium node for chrome*/
                sh "docker run -d --rm --net=SE --name app-test-container roci0055/frontend-calculator" /* container running the actual application */
                /* they are all added to the same network within docker they are able to communicate */
            }
        }
        stage("Execute system tests") { /*Final stage: to execute the tests by running the command selenium-side-runner*/
            steps {
                sh "selenium-side-runner --server http://localhost:4444/wd/hub -c 'browserName=chrome --base-url http://app-test-container test/system/FunctionalTests.side"
            }
        }
    }
    post {
        cleanup { /* This will be run no matter what happens on the previous stages */
            echo "Cleaning the Docker environment"
            sh script:"docker stop selenium-hub", returnStatus:true
            sh script: "docker stop selenium-node-chrome", returnStatus:true
            sh script: "docker stop app-test-container", returnStatus:true
            sh script: "docker network remove SE", returnStatus:true
        }
    }
}