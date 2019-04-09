pipeline {
    agent any
    environment {
        BUILD_CAUSE = "${env.BUILD_CAUSE}"
    }

    options {
        timestamps()
        buildDiscarder logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '', numToKeepStr: '20')
        ansiColor('xterm')
        skipDefaultCheckout true
    }
    stages {

        stage( 'User Input'){
            steps{
                    input('Do you want to proceed ?')
                }
        }

        stage ( 'Downloading Source Code' ){
            //agent any
            steps{
                deleteDir()
                checkout([$class: 'GitSCM', branches: [[name: '*/develop'],[name: '${sha1}']], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'SparseCheckoutPaths', sparseCheckoutPaths: [[path: 'apmonshore/apmonshore-well-commissioning'], [path: 'config/settings.xml']]], [$class: 'PathRestriction', excludedRegions: '', includedRegions: 'apmonshore/apmonshore-well-commissioning/.*']], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'FINAL', url: 'https://github.build.ge.com/503078266/APM-Onshore.git']]])
                //checkout([$class: 'GitSCM', branches: [[name: '*/jenkinsfile'], [name: '*/develop'],[name: '${sha1}']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'FINAL', url: 'https://github.build.ge.com/503078266/APM-Onshore.git']]])
                }
        }

        stage('Building APM Onshore Well Commissioning ') {
            //agent any
            steps {
                  //deleteDir()
                  sh 'ls -lrt'
                  sh 'cd $WORKSPACE/apmonshore/apmonshore-well-commissioning/'
                  sh 'pwd'
                  sh '/opt/apache-maven-3.3.9/bin/mvn -f apmonshore/apmonshore-well-commissioning/pom.xml  clean install test sonar:sonar -e -s $WORKSPACE/config/settings.xml'
            }
        }

        stage('Maven Cleanup & ZIP file') {
            //agent any
            steps{
                sh '''
                    echo "Cleaning up maven files"
                    rm -rf /home/jenkins/.m2/repository/com/ge/apmonshore/apmonshore/
                    rm -rf /home/jenkins/.m2/repository/com/ge/apmonshore/commonutils/apmonshore-commonutils/
                    rm -rf /home/jenkins/.m2/repository/com/ge/apmonshore/multitenancy/
                    echo "Creating ZIP file"
                    cd $WORKSPACE/apmonshore/apmonshore-well-commissioning/
                    zip -r apmonshore-well-commissioning_1.0.0.$BUILD_NUMBER.zip manifest*.yml target/*
                '''

            }
        }

        /*stage('Create workspace link') {
            agent { node { label 'master' } }
            steps {
                def Foldername = jenkinsfile;
                def theString = "<a href='https://jenkins.com/job/" + Foldername + "/" + BUILD_NUMBER + "/execution/node/3/ws/'>Workspace</a>";
                manager.addShortText(theString, "blue", "white", "0px", "white");
                manager.createSummary("green.gif").appendText("<h1>" + theString + "</h1>", false, false, false, "blue");
            }
        }*/

        stage('Upload Artifacts'){
           // agent any
            steps {
                echo 'We have checked and uploding artifact is working fine...'
                //sh 'curl -X PUT -u 502712493:AP3QVE55ZDfXm8giwQn6JSFfem6  https://devcloud.swcoe.ge.com/artifactory/list/APM-Onshore/APMOnshore/Services/well-commissioning01/latest/uploading_from_pipeline.txt --upload-file $WORKSPACE/apmonshore/apmonshore-well-commissioning/apmonshore-well-commissioning_1.0.0.$BUILD_NUMBER.zip'
            }
        }
        //stage(Download Artifacts){
          //   if ("${$BUILD_CAUSE}" == "9.2.0") {
        //}

    }
    post {
        success {
            echo 'Calling downstream jobs!'

            echo '1. Coverity'
            //coverityResults connectInstance: 'Coverity', connectView: 'Onshore Microservices Stream', projectId: 'Onshore Microservices'

            echo '2. First Downstream Job'
            build job: 'upload', parameters:
            [
                string(name: 'Build', value: "${BUILD_NUMBER}"),

            ]

            echo '3. Second Downstream Job'
            build job: 'DSL-Test'

            echo '4. Mail'
            //emailext attachLog: true, body: 'Please find attachment', compressLog: true, subject: '$Build_Number Status', to: 'Kundan.Thakur@bhge.com'
            //mail bcc: '', body: 'PFA', cc: '', from: '', replyTo: '', subject: 'Build Status ', to: 'Kundan.Thakur@bhge.com'
            step([$class: 'Mailer', notifyEveryUnstableBuild: true, recipients: 'Kundan.Thakur@bhge.com', sendToIndividuals: false])

        }
    }
}
