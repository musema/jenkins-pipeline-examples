
pipeline {
    agent any
    tools {
        maven "Maven-3.3.9"
        jdk "Java-8"
    }
    environment{
        DEVELOPERS_EMAIL="musema.hassen@gmail.com"
        BUILD_FROM_BRANCH="master"
        REPO_URL="https://github.com/musema/jenkins-pipeline-examples.git"
    }
    options{
        timeout(time:48,unit:'HOURS')
        skipDefaultCheckout(true)
    }
    stages {
        stage('Prepare'){
            parallel{
                stage('Code-Checkout'){
                    steps{
                       checkoutCode()
                    }
                }
                stage('Prepare-Workspace'){
                    steps{
                        prepareWorkspace()
                    }
                }
            }
        }
        stage("Package") {
            steps {
                packageArtifact()
            }
        }
        stage("Test"){
            steps{
                runTest()
            }
        }
        stage("Upload to Repository"){
            steps{
                uploadToRepository();
            }
        }
        stage("Decide-Deployment"){
            agent none
            steps{
                script{
                    env.DEPLOY_TO_DEV= input message: 'Approval is required',
							parameters: [
                                choice(name: 'Do you want to deploy to DEV?', choices: 'no\nyes', 
                                description: 'Choose "yes" if you want to deploy the DEV server')
                            ]
                }
            }
        }
        stage('Deploy To Dev'){
            when{
                environment name:'DEPLOY_TO_DEV', value:'yes'
            }
            steps{
                deployToServer("DEV")
            }
        }
        stage('Archive'){
            steps{
                archiveTheBuild()
            }
        }
    }
    post {
        always {
            finalizeWorkflow()

        }
            success {
                successMessage()
                 }
        failure {
                failureMessage()
            }
    }
}

//define groovy functions here
def prepareWorkspace(){
    echo 'Check here if everything is ready to make a build'
    sh 'mvn --version'
    sh 'java -version'
    sh 'git --version'
}
def checkoutCode(){
    count=1
    retry(3){
        echo "trying to checkout the code from SCM, Trial:${count}"
        //git branch: "${BUILD_FROM_BRANCH}",credentialsId: "${GITHUB_CREDENTIALS_ID}",url: "${REPO_URL}"
        git url:"${REPO_URL}"
    }
}
def packageArtifact(){
    sh 'mvn clean install -Dmaven.test.failure.ignore=true'
}
def runTest(){
    sh 'mvn test'
}
def uploadToRepository(){
    echo "We are about to publish artifacts to remote repository"
    //mvn deploy
}
def archiveTheBuild(){
    archive '*/target/**/*'
    //junit '*/target/surefire-reports/*.xml'

}
def deployToServer(deployTo){
    echo "We are deploying to : ${deployTo}"
}
def successMessage(){
    echo "Sending SUCCESS email to ${env.DEVELOPERS_EMAIL}"
    notifyBuild('SUCCESS')
}
def failureMessage(){
    echo "Notifying build failure"
    notifyBuild('FAILED')
}
def finalizeWorkflow(){
    echo "Cleaning up the workspace"
}

def notifyBuild(bStatus) {
    def subject = "${bStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
    def details = """<html><body><p>STARTED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
    <p>Check console output at &QUOT;<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>&QUOT;</p></body></html>"""

    mail to:"${env.DEVELOPERS_EMAIL}",subject:"${subject}",body:"${details}"
}

