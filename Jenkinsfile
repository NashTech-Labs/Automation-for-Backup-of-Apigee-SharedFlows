import java.text.SimpleDateFormat

pipeline {
    agent any
    environment {
          GCLOUD_DIR = "$JENKINS_HOME/google-cloud-sdk/bin"
          APIGEE_CLI_DIR = "$HOME/.apigeecli/bin"
          GITHUB_CREDS = "<GITHUB_CREDS_ID>"
    }
    stages {
        stage('Installing Dependencies') {
            steps {
                sh '''#!/bin/bash
                    echo "Checking for pre-installed dependencies..."
                    echo ""
                    if [ ! -d "$GCLOUD_DIR" ]; then
                        echo "Installing GCloud CLI..."
                        echo ""
                        cd $JENKINS_HOME
                        curl -O https://dl.google.com/dl/cloudsdk/channels/rapid/downloads/google-cloud-cli-412.0.0-linux-x86_64.tar.gz
                        tar -xf google-cloud-cli-412.0.0-linux-*.tar.gz
                        ./google-cloud-sdk/install.sh -q
                        source $JENKINS_HOME/google-cloud-sdk/completion.bash.inc
                        source $JENKINS_HOME/google-cloud-sdk/path.bash.inc
                    else
                        echo "GCloud CLI is already Installed!"
                        echo ""
                    fi

                    if [ ! -d "$APIGEE_CLI_DIR" ]; then
                        echo "Installing Apigee CLI..."
                        echo ""
                        curl -L https://raw.githubusercontent.com/apigee/apigeecli/main/downloadLatest.sh | sh -
                    else
                        echo "Apigee CLI is already Installed!"
                        echo ""
                    fi
                '''
            }
        }
        stage('Logging into Google Cloud and Get Access Token') {
            steps {
                script {
                    withCredentials([file(credentialsId: '<GCP_CREDENTIAL_ID>', variable: 'GOOGLE_SERVICE_ACCOUNT_KEY')]) {
                        sh "${GCLOUD_DIR}/gcloud auth activate-service-account --key-file ${GOOGLE_SERVICE_ACCOUNT_KEY}"
                        env.TOKEN = sh([script: "${GCLOUD_DIR}/gcloud auth print-access-token", returnStdout: true]).trim()
                    }
                }
            }
        }
        stage("Clone Git Repository") {
            steps {
                git(
                    url: "https://github.com/<GITHUB_USER_NAME>/Apigee.git",
                    credentialsId: "${GITHUB_CREDS}",
                    branch: "main",
                    changelog: true,
                    poll: true
                )
            }
        }
       stage("Exporting sharedflows from Apigee") {
            steps {
                script {
                    def currentDate = new Date().format("dd-MM-yyyy")
                    def exportDir = "<APIGEE_ORG_NAME>/${currentDate}/sharedflows"
                    
                    sh """
                    mkdir -p ${exportDir}
                    ${APIGEE_CLI_DIR}/apigeecli sharedflows export -o <APIGEE_ORG_NAME> -t ${TOKEN} -f ${exportDir}
                    """
                    
                    sh "git config --global user.name '<GITHUB_USER_NAME>'"
                    sh "git config --global user.email '<GITHUB_USER_EMAIL>'"
                    sh "git add ."
                    
                    def changes = sh(script: "git status --porcelain", returnStdout: true).trim()
                    if (changes) {
                        sh "git commit -m 'Added sharedflows from Jenkins Pipeline'"
                        withCredentials([gitUsernamePassword(credentialsId: "${GITHUB_CREDS}", gitToolName: 'Default')]) {
                            sh "git push -u origin main"
                        }
                    } else {
                        echo "No changes to commit in sharedflows. Skipping git push."
                    }
                }
            }
        }
           stage("Cleanup data of month ago") {
                steps {
                    script {
                    def currentDate = new Date()
                    def thirtyDaysAgo = new Date() - 30
                      
                    sh "git config --global user.name '<GITHUB_USER_NAME>'"
                    sh "git config --global user.email '<GITHUB_USER_EMAIL>'"

                    def workspace = env.WORKSPACE
                    def dirsToDelete = []

                    def dirList = sh(script: "ls -d <APIGEE_ORG_NAME>/*", returnStdout: true).trim().split("\n")
                    dirList.each { dir ->
                    def dirDate = dir.replaceAll(".*<APIGEE_ORG_NAME>/", "")
                    def sdf = new SimpleDateFormat("dd-MM-yyyy")
                    try {
                        def dirTimestamp = sdf.parse(dirDate).time

                        if (dirTimestamp <= thirtyDaysAgo.time) {
                        dirsToDelete.add(dir)
                        }
                    } catch (java.text.ParseException e) {
                    }
                }

                    dirsToDelete.each { dir ->
                    sh "rm -rf ${dir}"
                    }

                    sh "git add -A" 
                    def changes = sh(script: "git status --porcelain", returnStdout: true).trim()
                    if (changes) {
                        sh "git commit -m 'Cleanup data Jenkins Pipeline'"
                        withCredentials([gitUsernamePassword(credentialsId: "${GITHUB_CREDS}", gitToolName: 'Default')]) {
                        sh "git push -u origin main"
                    }
                    } else {
                        echo "No changes in last one month to commit. Skipping git push."
                    }
                }
            }
        }
    }

    post {
        always {
            deleteDir()
        }
    }

}