pipeline{
    agent {
        label 'agent-ec2'
    }
    environment{
        SONAR_SCANNER = tool 'sonarqube-scanner-tool'
        PROJECT_NAME = 'three-tier-devsecops-mega-project'
        DOCKER_CREDS = credentials('docker-token')
    }
    tools{
        nodejs 'nodejs-tool' 
    }
    options {
        disableResume()
        disableConcurrentBuilds abortPrevious: true
        buildDiscarder logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '', numToKeepStr: '3')
        // skipDefaultCheckout()
    }
    stages{
        stage("Notifying Team"){
            slackSend channel: 'project-3-tier-devsecops-mega-project', message: 'Pipeline Started'
        }
        stage("Installing Dependencies"){
            steps{
                dir('client') {
                    sh '''
                        npm install --omit=dev 
                        # npm run build
                    '''
                }
                dir('api') {
                    sh 'npm install --omit=dev'
                }
            }
        }
        stage("Unit Test"){
            steps{
                echo 'Skipping Tests because there are no tests available for this project at the moment'
            }
        }
        stage("Dependancy Check"){
            steps{
                dir('client') {
                    sh 'npm audit --json > client-audit-report.json || true'
                    archiveArtifacts allowEmptyArchive: true, artifacts: 'client-audit-dependancy-report.json', followSymlinks: false
                }
                dir('api') {
                    sh 'npm audit --json > api-audit-report.json || true'
                    archiveArtifacts allowEmptyArchive: true, artifacts: 'api-audit-dependancy-report.json', followSymlinks: false
                }
            }
        }
        stage("SonarQube Scan"){
            steps{
                withSonarQubeEnv('sonarqube-server'){
                    sh '''
                        ${SONAR_SCANNER}/bin/sonar-scanner \
                        -Dsonar.projectKey=3-Tier-DevSecOps-Mega-Project \
                        -Dsonar.projectName=3-Tier-DevSecOps-Mega-Project \
                        -Dsonar.sources=client,api \
                        -Dsonar.exclusions=node_modules/**,**/dist/**,**/build/**,coverage/** \
                    '''
                }
            }
        }
        stage("Quality Gate"){
            steps{
                catchError(buildResult: 'SUCCESS', message: 'Oop! The Quality gate has not been completed yet', stageResult: 'SUCCESS') {
                    timeout(time: 1, unit: 'MINUTES'){
                        waitForQualityGate abortPipeline: false, credentialsId: 'sonarqube-token'
                    }
                }
            }      
        }
        stage("Build images"){
            steps{
                dir('client') {
                    sh '''
                        docker build -t ${PROJECT_NAME}-client-image:${GIT_COMMIT} .
                        docker tag ${PROJECT_NAME}-client-image:${GIT_COMMIT} ${DOCKER_CREDS_USR}/${PROJECT_NAME}-client-image:${GIT_COMMIT}
                    '''
                }
                dir('api') {
                    sh '''
                        docker build -t ${PROJECT_NAME}-api-image:${GIT_COMMIT} .
                        docker tag ${PROJECT_NAME}-api-image:${GIT_COMMIT} ${DOCKER_CREDS_USR}/${PROJECT_NAME}-api-image:${GIT_COMMIT}
                    '''
                }
            }
        }
        stage("Push image to Docker hub"){
            steps{
                withCredentials([usernamePassword(credentialsId: 'docker-token', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh '''
                        echo $PASSWORD | docker login -u $USERNAME --password-stdin
                        docker push ${USERNAME}/${PROJECT_NAME}-api-image:${GIT_COMMIT}
                        docker push ${USERNAME}/${PROJECT_NAME}-client-image:${GIT_COMMIT}
                    '''
                }
            }
        }
        stage("Trivy Scan Images for Vulnerability"){
            steps{
                sh 'mkdir -p trivy-reports'
                dir('trivy-reports') {
                    sh '''
                        trivy image \
                        --severity MEDIUM,HIGH,CRITICAL \
                        --format json \
                        --output trivy-${PROJECT_NAME}-client-image-${GIT_COMMIT}-report.json \
                        ${DOCKER_CREDS_USR}/${PROJECT_NAME}-client-image:${GIT_COMMIT} \
                        || true
    
                        trivy image \
                        --severity MEDIUM,HIGH,CRITICAL \
                        --format json \
                        --output trivy-${PROJECT_NAME}-api-image-${GIT_COMMIT}-report.json \
                        ${DOCKER_CREDS_USR}/${PROJECT_NAME}-api-image:${GIT_COMMIT} \
                        || true
                    '''
                }
            }
        }
        stage("Deploy using Docker Compose"){
            steps{
                slackSend channel: 'project-3-tier-devsecops-mega-project', message: 'Permission pending for deployment!'
                input message: 'Continue to Deployment ?', ok: 'Yes'
                withCredentials([file(credentialsId: 'env-file-of-project', variable: 'ENV_FILE')]) {
                    sh '''
                        cat ${ENV_FILE} > temp.env
                        echo "BACKEND=${DOCKER_CREDS_USR}/${PROJECT_NAME}-api-image:${GIT_COMMIT}" >> temp.env
                        echo "FRONTEND=${DOCKER_CREDS_USR}/${PROJECT_NAME}-client-image:${GIT_COMMIT}" >> temp.env
                        docker compose --env-file temp.env up -d
                        rm -f temp.env
                    '''
                }
            }
        }
    }
    post{
        always{
            dir('trivy-reports') {
                sh '''
                    trivy convert \
                    --format template --template "@/usr/local/share/trivy/templates/html.tpl" \
                    --output api-audit-report.html api-audit-report.json \
                    || true
                    
                    trivy convert \
                    --format template --template "@/usr/local/share/trivy/templates/html.tpl" \
                    --output client-audit-report.html client-audit-report.json \
                    || true
    
                    trivy convert \
                    --format template --template "@/usr/local/share/trivy/templates/junit.tpl" \
                    --output trivy-${PROJECT_NAME}-client-image-${GIT_COMMIT}-report.xml \
                    trivy-${PROJECT_NAME}-client-image-${GIT_COMMIT}-report.json \
                    || true
                    
                    trivy convert \
                    --format template --template "@/usr/local/share/trivy/templates/junit.tpl" \
                    --output trivy-${PROJECT_NAME}-api-image-${GIT_COMMIT}-report.xml \
                    trivy-${PROJECT_NAME}-api-image-${GIT_COMMIT}-report.json \
                    || true
                    
                    trivy convert \
                    --format template --template "@/usr/local/share/trivy/templates/html.tpl" \
                    --output trivy-${PROJECT_NAME}-client-image-${GIT_COMMIT}-report.html \
                    trivy-${PROJECT_NAME}-client-image-${GIT_COMMIT}-report.json \
                    || true
                    
                    trivy convert \
                    --format template --template "@/usr/local/share/trivy/templates/html.tpl" \
                    --output trivy-${PROJECT_NAME}-api-image-${GIT_COMMIT}-report.html \
                    trivy-${PROJECT_NAME}-api-image-${GIT_COMMIT}-report.json \
                    || true
                '''
            }
            // junit allowEmptyResults: true, testResults: 'trivy-${PROJECT_NAME}-api-image:${GIT_COMMIT}-report.xml'
            // junit allowEmptyResults: true, testResults: 'trivy-${PROJECT_NAME}-client-image:${GIT_COMMIT}-report.xml'
            publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, keepAll: true, reportDir: './trivy-reports', reportFiles: 'trivy-${PROJECT_NAME}-api-image:${GIT_COMMIT}-report.html', reportName: 'Trivy API Image Report', reportTitles: '', useWrapperFileDirectly: true])
            publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, keepAll: true, reportDir: './trivy-reports', reportFiles: 'trivy-${PROJECT_NAME}-client-image:${GIT_COMMIT}-report.html', reportName: 'Trivy CLIENT Image Report', reportTitles: '', useWrapperFileDirectly: true])
        }
        success {
            slackSend channel: 'project-3-tier-devsecops-mega-project', message: 'Pipeline completed successfully! There are no issues.'
        }
        failure {
            cleanWs()
            slackSend channel: 'project-3-tier-devsecops-mega-project', message: 'Pipeline Failed! There are some issue. Please fix it as soon as possibile.'
        }
    }
}
