#!groovy

PROJECT_NAME = "eureka-server-app"
PROJECT_KEY = "tiggomark"
SCM = "GITHUB"

def repositoryUrl = 'tiggomark'

node {

   // def javaHome = tool 'java11'
    //env.PATH = "${javaHome}/bin:${env.PATH}"
   // env.JAVA_HOME = "${javaHome}"

    def branch = "master"
    def environment = "prod"
    def tagVersion = "dev"

    stage(name: 'Checkout from SCM') {
        echo 'Checking out from scm'
        checkout scm
    }

    stage(name: 'Gradle build') {
        echo 'Testing project'
        sh 'chmod +x gradlew'
        sh "./gradlew clean build --stacktrace"
    }

    stage(name: 'Build docker image') {
        echo 'Build docker image and push to registry'
        withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
            sh "docker login -u $USERNAME -p $PASSWORD"
            pomVersion = getVersion()
            if(environment == "qa") {
                sh "docker build -f ./Dockerfile --build-arg VERSION=$pomVersion --build-arg APP=$PROJECT_NAME -t ${repositoryUrl}/$PROJECT_NAME:$tagVersion ."
                echo "Build complete for version $pomVersion develop release...Upload image to Harbor"
                sh "docker push ${repositoryUrl}/$PROJECT_NAME:$tagVersion"
            } else {
                tagVersion = getVersion()
                echo "Initializing build release ${tagVersion}"
                sh "docker build -f ./Dockerfile --build-arg VERSION=$pomVersion --build-arg APP=$PROJECT_NAME -t ${repositoryUrl}/$PROJECT_NAME:$tagVersion -t ${repositoryUrl}/$PROJECT_NAME:latest ."
                echo "Build complete for version ${tagVersion} and latest release...Upload image to Harbor"
                sh "docker push ${repositoryUrl}/$PROJECT_NAME:$tagVersion && docker push ${repositoryUrl}/$PROJECT_NAME:latest"
            }
        }
    }

    stage(name: 'Deploy to Docker Container') {
        echo 'Deploying images to docker container'
        sh "docker rm --force $PROJECT_NAME"
        sh "docker run --network cluster-network -p 8761:8761 --name $PROJECT_NAME  -d  ${repositoryUrl}/$PROJECT_NAME:$tagVersion"
        echo "Deploy de ${PROJECT_NAME} para o ambiente ${environment} finalizado com sucesso"
        //sendMsgToSlack("Deploy de ${PROJECT_NAME} para o ambiente ${environment} finalizado com sucesso")
        currentBuild.result = "SUCCESS"
    }

}



def sendMsgToSlack(text) {

/*
    try {
        build(job: "send-slack-message",
                parameters: [
                        string(name: "TITLE", value: "[${PROJECT_NAME}][${branch}]"),
                        string(name: "TITLELINK", value: env.BUILD_URL),
                        string(name: "TEXT", value: text),
                        string(name: "CHANNEL", value: "#podolsk-alerts")
                ]
        )
    } catch (all) {
        echo "Não foi possível enviar a mensagem para o canal #podolsk-alerts"
        echo "Error: ${all.message}"
    }
    */
}

def getVersion() {
    def lines = readFile("${env.WORKSPACE}/gradle.properties").split("\n")

    return lines.find { it.contains("version") }
            .split("=")[1]
            .trim()
}