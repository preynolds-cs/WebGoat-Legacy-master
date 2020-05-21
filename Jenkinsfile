pipeline {
    agent any

    environment {
        ARTEFACT_NAME = "${WORKSPACE}/target/WebGoat-${BUILD_VERSION}.war"
        DEV_REPO = 'maven-development'
        TAG_FILE = "${WORKSPACE}/tag.json"
        IQ_SCAN_URL = ""
    }
    tools {
       maven 'M3'
    }
    stages {
        stage('Build') {
            steps {
                sh 'mvn -B -Dproject.version=$BUILD_VERSION -Dmaven.test.failure.ignore clean package'
            }
            post {
                success {
                    echo 'Now archiving ...'
                    archiveArtifacts artifacts: "**/target/*.war"
                }
            }
        }

        // Once you run this pipeline once, you will need to approve the script from the console output
        stage('Nexus IQ Scan'){
            steps {
                script{         
                    try {
                        def policyEvaluation = nexusPolicyEvaluation failBuildOnNetworkError: true, iqApplication: selectedApplication('Webgoat'), iqScanPatterns: [[scanPattern: '**/*.war']], iqStage: 'build', jobCredentialsId: ''
                        echo "Nexus IQ scan succeeded: ${policyEvaluation.applicationCompositionReportUrl}"
                        IQ_SCAN_URL = "${policyEvaluation.applicationCompositionReportUrl}"
                    } 
                    catch (error) {
                        def policyEvaluation = error.policyEvaluation
                        echo "Nexus IQ scan vulnerabilities detected', ${policyEvaluation.applicationCompositionReportUrl}"
                        throw error
                    }
                }
            }
        }

        stage('Create tag'){
            steps {
                script {
    
                    // Git data (Git plugin)
                    echo "url is ${GIT_URL}"
                    echo "${GIT_BRANCH}"
                    echo "why are you not printing - $USER "
                    echo "${GIT_COMMIT}"
                    echo "workspace is ${WORKSPACE}"

                    
                    // construct the meta data (Pipeline Utility Steps plugin)
                    // In Jenkins > Settings > Manage Plugins and install the "Pipeline utility" plugin << REQUIRED!!!
                   
                }
            }
        }

        stage('Upload to Nexus Repository'){
            steps {
                script {
                    nexusPublisher nexusInstanceId: 'nexus', nexusRepositoryId: "${DEV_REPO}", packages: [[$class: 'MavenPackage', mavenAssetList: [[classifier: '', extension: 'war', filePath: "${ARTEFACT_NAME}"]], mavenCoordinate: [artifactId: 'WebGoat', groupId: 'WebGoat', packaging: 'war', version: "${BUILD_VERSION}"]]], tagName: "${BUILD_TAG}"
                }
            }
        }
    }
}
