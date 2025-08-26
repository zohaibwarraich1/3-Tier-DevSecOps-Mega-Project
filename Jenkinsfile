pipeline{
    agent {
        label 'agent-ec2'
    }
    // environment{
    //     SONAR_SCANNER = tool 'sonarqube-scanner-tool'
    //     PROJECT_NAME = 'three-tier-devsecops-mega-project'
    //     DOCKER_CREDS = credentials('docker-token')
    // }
    // tools{
    //     nodejs 'nodejs-tool' 
    // }
    options {
        disableResume()
        disableConcurrentBuilds abortPrevious: true
        buildDiscarder logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '', numToKeepStr: '3')
        // skipDefaultCheckout()
    }
    stages{
        stage("Checkout Code"){
            steps{
                // git branch: 'main', url: 'https://github.com/zohaibwarraich1/3-Tier-DevSecOps-Mega-Project.git'
                sh '${GIT_COMMIT}'
            }
        }
    //     stage("Installing Dependencies"){
    //         steps{
    //             dir('client') {
    //                 sh '''
    //                     npm install --omit=dev 
    //                     # npm run build
    //                 '''
    //             }
    //             dir('api') {
    //                 sh 'npm install --omit=dev'
    //             }
    //         }
    //     }
    //     stage("Unit Test"){
    //         steps{
    //             echo 'Skipping Tests because there are no tests available for this project at the moment'
    //         }
    //     }
    //     stage("Dependancy Check"){
    //         steps{
    //             dir('client') {
    //                 sh 'npm audit --json > client-audit-report.json || true'
    //                 archiveArtifacts allowEmptyArchive: true, artifacts: 'client-audit-dependancy-report.json', followSymlinks: false
    //             }
    //             dir('api') {
    //                 sh 'npm audit --json > api-audit-report.json || true'
    //                 archiveArtifacts allowEmptyArchive: true, artifacts: 'api-audit-dependancy-report.json', followSymlinks: false
    //             }
    //         }
    //     }
    //     stage("SonarQube Scan"){
    //         steps{
    //             withSonarQubeEnv('sonarqube-server'){
    //                 sh '''
    //                     ${SONAR_SCANNER}/bin/sonar-scanner \
    //                     -Dsonar.projectKey=3-Tier-DevSecOps-Mega-Project \
    //                     -Dsonar.projectName=3-Tier-DevSecOps-Mega-Project \
    //                     -Dsonar.sources=client,api \
    //                     -Dsonar.exclusions=node_modules/**,**/dist/**,**/build/**,coverage/** \
    //                 '''
    //             }
    //         }
    //     }
    //     stage("Quality Gate"){
    //         steps{
    //             catchError(buildResult: 'SUCCESS', message: 'Oop! The Quality gate has not been completed yet', stageResult: 'ABORTED') {
    //                 timeout(time: 1, unit: 'MINUTES'){
    //                     waitForQualityGate abortPipeline: false, credentialsId: 'sonarqube-token'
    //                 }
    //             }
    //         }      
    //     }
    //     stage("Build images"){
    //         steps{
    //             dir('api') {
    //                 sh '''
    //                     docker build -t ${PROJECT_NAME}-client-image:${GIT_COMMIT} .
    //                     docker tag ${PROJECT_NAME}-client-image:${GIT_COMMIT} ${DOCKER_CREDS_USR}/${PROJECT_NAME}-client-image:${GIT_COMMIT}
    //                 '''
    //             }
    //             dir('api') {
    //                 sh '''
    //                     docker build -t ${PROJECT_NAME}-api-image:${GIT_COMMIT} .
    //                     docker tag ${PROJECT_NAME}-api-image:${GIT_COMMIT} ${DOCKER_CREDS_USR}/${PROJECT_NAME}-api-image:${GIT_COMMIT}
    //                 '''
    //             }
    //         }
    //     }
    //     stage("Push image to Docker hub"){
    //         steps{
    //             withCredentials([usernamePassword(credentialsId: 'docker-token', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
    //                 sh '''
    //                     echo $PASSWORD | docker login -u $USERNAME --password-stdin
    //                     docker push ${USERNAME}/${PROJECT_NAME}-api-image:${GIT_COMMIT}
    //                     docker push ${USERNAME}/${PROJECT_NAME}-client-image:${GIT_COMMIT}
    //                 '''
    //             }
    //         }
    //     }
    //     stage("Trivy Scan Images for Vulnerability"){
    //         steps{
    //             sh '''
    //                 trivy image \
    //                 --severity MEDIUM,HIGH,CRITICAL \
    //                 --format template \
    //                 --template "@contrib/html.tpl" \
    //                 --output trivy-${PROJECT_NAME}-client-image:${GIT_COMMIT}-report.html \
    //                 ${USERNAME}/${PROJECT_NAME}-client-image:${GIT_COMMIT}

    //                 trivy image \
    //                 --severity MEDIUM,HIGH,CRITICAL \
    //                 --format template \
    //                 --template "@contrib/html.tpl" \
    //                 --output trivy-${PROJECT_NAME}-api-image:${GIT_COMMIT}-report.html \
    //                 ${USERNAME}/${PROJECT_NAME}-api-image:${GIT_COMMIT}
    //             '''
    //         }
    //     }
    //     stage("Deploye using Docker Compose"){
    //         steps{
    //             withCredentials([file(credentialsId: 'env-file-of-project', variable: 'ENV_FILE')]) {
    //                 sh 'docker compose --env-file ${ENV_FILE} up -d'
    //             }
    //         }
    //     }
    }
    // post{
    //     always{
    //         cleanWs()
    //     }
    // }
}
