PipeLine Note
================

pipeline的 HTML Publisher Plugin使用
-----------------------------------------

.. code::

    pipeline {
        agent { label 'master' }
        stages{
            stage('Firmware_Reports'){
                steps{
                    publishHTML (target: [
                        allowMissing: false,
                        alwaysLinkToLastBuild: false,
                        keepAll: true,
                        reportDir: 'Firmware_Reports',
                        reportFiles: 'log.html',
                        reportName: "Firmware Html Report"])
                }
            }
            stage('Software_Reports'){
                steps{
                    publishHTML (target: [
                        allowMissing: false,
                        alwaysLinkToLastBuild: false,
                        keepAll: true,
                        reportDir: 'Software_Reports',
                        reportFiles: 'log.html',
                        reportName: "Software Html Report"])
                }
            }
        }
    }

配置实例
---------------------------------

.. code::

    pipeline {
        agent { label "master" }

        environment {
            TRIGGER_NUM = "${BUILD_NUMBER}"
            GITLAB_BRANCH = "${gitlabSourceBranch}"
            GITLAB_NAME = "${gitlabSourceRepoName}"
            GITLAB_COMMITER = "${gitlabUserEmail}"
            GITLAB_COMMIT = "${gitlabAfter}"
            GITLAB_BEFORE = "${gitlabBefore}"
            GITLAB_HTTPURL = "${gitlabSourceRepoHttpUrl}"
            GITLAB_GROUP = "${gitlabSourceNamespace}"
        }

        options { 
            timestamps()
            timeout(time: 2, unit: 'HOURS')
        }
        
        stages {
            stage("Compile And Test") {
                parallel {
                    stage("Master") {
                        stages {
                            stage("Prepare") {
                                steps {
                                    script {
                                        currentBuild.displayName = "${BUILD_NUMBER} -> ${gitlabSourceRepoName}"
                                        try {
                                            sh "mkdir -m 777 -p ${JENKINS_HOME}/userContent/${GITLAB_GROUP}/${GITLAB_NAME}/${TRIGGER_NUM}"
                                        } catch (err) {
                                            echo "Caught error in mkdir on master: ${err}"
                                        }
                                        try {
                                            sh "rm -rf ${WORKSPACE}/*"
                                        } catch (err) {
                                            echo "Caught error in clean ${WORKSPACE} on master: ${err}"
                                        }
                                    }
                                    checkout changelog: false, poll: false, 
                                        scm: [$class: "GitSCM", 
                                              branches: [[name: "${GITLAB_COMMIT}"]], 
                                              doGenerateSubmoduleConfigurations: false, 
                                              extensions: [[$class:"RelativeTargetDirectory",
                                                            relativeTargetDir:"${GITLAB_GROUP}/${GITLAB_NAME}"],
                                                           [$class: "CleanCheckout"]], 
                                              submoduleCfg: [], 
                                              userRemoteConfigs: [[credentialsId: "123", 
                                                                   url: "${GITLAB_HTTPURL}"]]]
                                    script {
                                        Test_Firmware = "No"
                                        Test_Software = "No"
                                        echo '---------------------------------0'
                                        if (fileExists("${GITLAB_GROUP}/${GITLAB_NAME}/02 ci/02 firmware/hlt/robotRun.py")) {
                                            Test_Firmware = "Yes"
                                            echo '---------------------------------1'
                                        }
                                        if (fileExists("${GITLAB_GROUP}/${GITLAB_NAME}/02 ci/03 software/hlt/robotRun.py")) {
                                            Test_Software = "Yes"
                                            echo '---------------------------------2'
                                        }
                                        echo '---------------------------------4'
                                    }
                                }
                            }
                        }
                    }
                    stage("Firmware") {
                        stages {
                            stage('Compile'){
                                agent { label "firmware_compile" }
                                steps {
                                    checkout changelog: false, poll: false, 
                                        scm: [$class: "GitSCM", 
                                              branches: [[name: "${GITLAB_COMMIT}"]], 
                                              doGenerateSubmoduleConfigurations: false, 
                                              extensions: [[$class:"RelativeTargetDirectory",
                                                            relativeTargetDir:"${GITLAB_GROUP}/${GITLAB_NAME}"],
                                                           [$class: "CleanCheckout"]], 
                                              submoduleCfg: [], 
                                              userRemoteConfigs: [[credentialsId: "123", 
                                                                   url: "${GITLAB_HTTPURL}"]]]

                                    dir("${WORKSPACE}/${GITLAB_GROUP}/${GITLAB_NAME}") {
                                        script {
                                            try {
                                                bat("call \"02 ci\\04 common\\common_script.bat\" compile firmware")
                                            } catch (err) {
                                                echo "Caught error in firmware compile: ${err}"
                                                currentBuild.result = 'FAILURE'
                                            } finally {
                                                bat("call \"02 ci\\04 common\\common_script.bat\" post compile firmware")
                                            }
                                        }
                                    }

                                    dir("${WORKSPACE}/${GITLAB_GROUP}/${GITLAB_NAME}") {
                                        script {
                                            try {
                                                bat("call \"02 ci\\04 common\\common_script.bat\" static firmware")
                                            } catch (err) {
                                                echo "Caught error in firmware static: ${err}"
                                                currentBuild.result = 'FAILURE'
                                            } finally {
                                                bat("call \"02 ci\\04 common\\common_script.bat\" post static firmware")
                                            }
                                        }
                                    }
                                }
                            }
                            stage("Test") {
                                when { equals expected: Test_Firmware, actual: "Yes" }
                                agent { label "firmware_test" }
                                steps {
                                    checkout changelog: false, poll: false, 
                                        scm: [$class: "GitSCM", 
                                              branches: [[name: "${GITLAB_COMMIT}"]], 
                                              doGenerateSubmoduleConfigurations: false, 
                                              extensions: [[$class:"RelativeTargetDirectory",
                                                            relativeTargetDir:"${GITLAB_GROUP}/${GITLAB_NAME}"],
                                                           [$class: "CleanCheckout"]], 
                                              submoduleCfg: [], 
                                              userRemoteConfigs: [[credentialsId: "123", 
                                                                   url: "${GITLAB_HTTPURL}"]]]

                                    dir("${WORKSPACE}/${GITLAB_GROUP}/${GITLAB_NAME}") {
                                        script {
                                            try {
                                                bat("call \"02 ci\\04 common\\common_script.bat\" test firmware")
                                            } catch (err) {
                                                echo "Caught error in firmware test: ${err}"
                                                currentBuild.result = 'FAILURE'
                                            } finally {
                                                bat("call \"02 ci\\04 common\\common_script.bat\" post test firmware")
                                            }
                                            if (fileExists("02 ci/02 firmware/hlt/Reports")) {
                                                step([
                                                    $class               : "RobotPublisher",
                                                    outputPath           : "02 ci/02 firmware/hlt/Reports",
                                                    outputFileName       : "*/output.xml",
                                                    reportFileName       : "report.html",
                                                    logFileName          : "log.html",
                                                    disableArchiveOutput : false,
                                                    passThreshold        : 100,
                                                    unstableThreshold    : 95.0,
                                                    otherFiles           : "*.txt",
                                                ])
                                            }
                                        }
                                    }
                                }
                            }
                        }
                    }
                    stage("Software") {
                        stages {
                            stage('Compile'){
                                agent { label "software_compile" }
                                steps {
                                    checkout changelog: false, poll: false, 
                                        scm: [$class: "GitSCM", 
                                              branches: [[name: "${GITLAB_COMMIT}"]], 
                                              doGenerateSubmoduleConfigurations: false, 
                                              extensions: [[$class:"RelativeTargetDirectory",
                                                            relativeTargetDir:"${GITLAB_GROUP}/${GITLAB_NAME}"],
                                                           [$class: "CleanCheckout"]], 
                                              submoduleCfg: [], 
                                              userRemoteConfigs: [[credentialsId: "123", 
                                                                   url: "${GITLAB_HTTPURL}"]]]

                                    dir("${WORKSPACE}/${GITLAB_GROUP}/${GITLAB_NAME}") {
                                        script {
                                            try {
                                                bat("call \"02 ci\\04 common\\common_script.bat\" compile software")
                                            } catch (err) {
                                                echo "Caught error in software compile: ${err}"
                                                currentBuild.result = 'FAILURE'
                                            } finally {
                                                bat("call \"02 ci\\04 common\\common_script.bat\" post compile software")
                                            }
                                        }
                                    }

                                    dir("${WORKSPACE}/${GITLAB_GROUP}/${GITLAB_NAME}") {
                                        script {
                                            try {
                                                bat("call \"02 ci\\04 common\\common_script.bat\" static software")
                                            } catch (err) {
                                                echo "Caught error in software static: ${err}"
                                                currentBuild.result = 'FAILURE'
                                            } finally {
                                                bat("call \"02 ci\\04 common\\common_script.bat\" post static software")
                                            }
                                        }
                                    }
                                }
                            }

                            stage("Test") {
                                when { equals expected: Test_Software, actual: "Yes" }
                                agent { label "software_test" }
                                steps {
                                    checkout changelog: false, poll: false, 
                                        scm: [$class: "GitSCM", 
                                              branches: [[name: "${GITLAB_COMMIT}"]], 
                                              doGenerateSubmoduleConfigurations: false, 
                                              extensions: [[$class:"RelativeTargetDirectory",
                                                            relativeTargetDir:"${GITLAB_GROUP}/${GITLAB_NAME}"],
                                                           [$class: "CleanCheckout"]], 
                                              submoduleCfg: [], 
                                              userRemoteConfigs: [[credentialsId: "123", 
                                                                   url: "${GITLAB_HTTPURL}"]]]
                            
                                    dir("${WORKSPACE}/${GITLAB_GROUP}/${GITLAB_NAME}") {
                                        script {
                                            try {
                                                bat("call \"02 ci\\04 common\\common_script.bat\" test software")
                                            } catch (err) {
                                                echo "Caught error in software test: ${err}"
                                                currentBuild.result = 'FAILURE'
                                            } finally {
                                                bat("call \"02 ci\\04 common\\common_script.bat\" post test software")
                                            }
                                            if (fileExists("02 ci/02 firmware/hlt/Reports")) {
                                                step(
                                                    [$class               : "RobotPublisher",
                                                    outputPath           : "02 ci/03 software/hlt/Reports",
                                                    outputFileName       : "*/output.xml",
                                                    reportFileName       : "report.html",
                                                    logFileName          : "log.html",
                                                    disableArchiveOutput : false,
                                                    passThreshold        : 100,
                                                    unstableThreshold    : 95.0,
                                                    otherFiles           : "*.txt",]
                                                )
                                            }
                                        }
                                    }
                                }
                            }
                        }
                    }
                }
            }
            stage("Collection Reports") {
                steps {
                    dir("${WORKSPACE}/${GITLAB_GROUP}/${GITLAB_NAME}") {
                        sh "\"02 ci/04 common/control.sh\""
                    }
                }
            }
        }
        post {
            success {
                emailext subject:"${GITLAB_NAME}, -Build #${TRIGGER_NUM} ${currentBuild.result}, commiter:${GITLAB_COMMITER}",
                         mimeType:'text/html',
                         body: '${FILE, path="summary.html"}', 
                         to: '${FILE,path="mail_list"} ${DEFAULT_RECIPIENTS}'
            }
            unstable {
                emailext subject:"${GITLAB_NAME}, -Build #${TRIGGER_NUM} ${currentBuild.result}, commiter:${GITLAB_COMMITER}",
                         mimeType:'text/html',
                         body: '${FILE, path="summary.html"}', 
                         to: '${FILE,path="mail_list"} ${DEFAULT_RECIPIENTS}'
            }
            failure {
                emailext subject:"${GITLAB_NAME}, -Build #${TRIGGER_NUM} ${currentBuild.result}, commiter:${GITLAB_COMMITER}",
                         mimeType:'text/html',
                         body: '${FILE, path="summary.html"}', 
                         to: '${FILE,path="mail_list"} ${DEFAULT_RECIPIENTS}'
            }
        }
    }

