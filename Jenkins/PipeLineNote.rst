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

when检查文件是否存在
-------------------------------

.. code::

    when { expression  { fileExists('cppcheck.xml') }}
    steps {
        publishCppcheck pattern:'cppcheck.xml',
                        displayErrorSeverity: true, 
                        displayNoCategorySeverity: true, 
                        displayPerformanceSeverity: true, 
                        displayPortabilitySeverity: true, 
                        displayStyleSeverity: true, 
                        displayWarningSeverity: true
    }

jenkins build 的结果反馈给gitlab
--------------------------------------

.. code::

    pipeline {
      agent any
     
      options {
        gitLabConnection('Your GitLab Connection')
      }

      stages {
        stage('build') {
          steps {
            updateGitlabCommitStatus name: 'build', state: 'running'
            hogehoge
          }
        }
      }
     
      post {
        success {
          updateGitlabCommitStatus name: 'build', state: 'success'
        }
        failure {
          updateGitlabCommitStatus name: 'build', state: 'failed'
        }
      }
    }

gitLabConnection: 是和GitLab连接的名称. 根据用户的权限可以在Job->configure->General->GitLab Connection看到具体的名称; 
而这个连接的名称是在Manage Jenkins->Configure System->Gitlab中配置的

updateGitlabCommitStatus: name - build 名称, 可以根据不同的步骤指定不同的名称, 就是一个普通字符串, 结果回传过后可以在
Gitlab中看到; state - 回传的状态, 包括: pending, running, canceled, success, failed

jenkins 官方接口：https://jenkins.io/doc/pipeline/steps/gitlab-plugin/#updategitlabcommitstatus-update-the-commit-status-in-gitlab

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
            gitLabConnection('tangke')
        }

        stages {
            stage("Prepare") {
                steps {
                    updateGitlabCommitStatus name: 'Prepare', state: 'running'
                    // 创建日志目录, 设置build名字, 清理workspace
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
                    // 下载代码
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
                    // 解析提交日志, 得出编译和测试条件, 并生成邮件列表
                    script {
                        sh "python \"${GITLAB_GROUP}/${GITLAB_NAME}/02 ci/04 common/analysis_gitlog.py\""
                        Compile_Firmware = "No"
                        Compile_Software = "No"
                        Test_Firmware = "No"
                        Test_Software = "No"

                        if (fileExists("${GITLAB_GROUP}\\${GITLAB_NAME}\\02 ci\\02 firmware\\hlt\\robotRun.py")) {
                            Test_Firmware = "Yes"
                        }
                        if (fileExists("${GITLAB_GROUP}\\${GITLAB_NAME}\\02 ci\\03 software\\hlt\\robotRun.py")) {
                            Test_Software = "Yes"
                        }

                        compile_list = readFile('compile_list')
                        if (compile_list.contains('Compile_Firmware=Yes')) {
                            Compile_Firmware = "Yes"
                        }
                        if (compile_list.contains('Compile_Software=Yes')) {
                            Compile_Software = "Yes"
                        }

                        if (Compile_Firmware == "No") {
                            Test_Firmware = "No"
                        }
                        if (Compile_Software == "No") {
                            Test_Software = "No"
                        }

                        println("Compile_Firmware: ${Compile_Firmware}")
                        println("Compile_Software: ${Compile_Software}")
                        println("Test_Firmware: ${Test_Firmware}")
                        println("Test_Software: ${Test_Software}")
                    }
                    updateGitlabCommitStatus name: 'Prepare', state: 'success'
                }
            }
            
            stage("Compile") {
                parallel {
                    stage("Firmware") {
                        when { equals expected: Compile_Firmware, actual: "Yes" }
                        stages {
                            stage('Firmware Compile') {
                                agent { label "firmware_compile" }
                                steps {
                                    updateGitlabCommitStatus name: 'FirmwareCompile', state: 'running'
                                    // 下载代码
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

                                    // 编译固件
                                    dir("${WORKSPACE}/${GITLAB_GROUP}/${GITLAB_NAME}") {
                                        script {
                                            try {
                                                bat("call \"02 ci\\04 common\\common_script.bat\" compile firmware")
                                            } catch (err) {
                                                echo "Caught error in firmware compile: ${err}"
                                                currentBuild.result = 'FAILURE'
                                                updateGitlabCommitStatus name: 'FirmwareCompile', state: 'failed'
                                            } finally {
                                                bat("call \"02 ci\\04 common\\common_script.bat\" post compile firmware")
                                            }
                                        }
                                    }
                                    
                                    // 静态解析固件代码
                                    dir("${WORKSPACE}/${GITLAB_GROUP}/${GITLAB_NAME}") {
                                        script {
                                            try {
                                                bat("call \"02 ci\\04 common\\common_script.bat\" static firmware")
                                            } catch (err) {
                                                echo "Caught error in firmware static: ${err}"
                                                currentBuild.result = 'FAILURE'
                                                updateGitlabCommitStatus name: 'FirmwareCompile', state: 'failed'
                                            } finally {
                                                bat("call \"02 ci\\04 common\\common_script.bat\" post static firmware")
                                            }
                                        }
                                    }
                                    updateGitlabCommitStatus name: 'FirmwareCompile', state: 'success'
                                }
                            }
                        }
                    }
                    stage("Software") {
                        when { equals expected: Compile_Software, actual: "Yes" }
                        stages {
                            stage('Software Compile') {
                                agent { label "software_compile" }
                                steps {
                                    updateGitlabCommitStatus name: 'SoftwareCompile', state: 'running'
                                    // 下载代码
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
       
                                    // 编译软件
                                    dir("${WORKSPACE}/${GITLAB_GROUP}/${GITLAB_NAME}") {
                                        script {
                                            try {
                                                bat("call \"02 ci\\04 common\\common_script.bat\" compile software")
                                            } catch (err) {
                                                echo "Caught error in software compile: ${err}"
                                                currentBuild.result = 'FAILURE'
                                                updateGitlabCommitStatus name: 'SoftwareCompile', state: 'failed'
                                            } finally {
                                                bat("call \"02 ci\\04 common\\common_script.bat\" post compile software")
                                            }
                                        }
                                    }
                                    
                                    // 静态解析软件代码
                                    dir("${WORKSPACE}/${GITLAB_GROUP}/${GITLAB_NAME}") {
                                        script {
                                            try {
                                                bat("call \"02 ci\\04 common\\common_script.bat\" static software")
                                            } catch (err) {
                                                echo "Caught error in software static: ${err}"
                                                currentBuild.result = 'FAILURE'
                                                updateGitlabCommitStatus name: 'SoftwareCompile', state: 'failed'
                                            } finally {
                                                bat("call \"02 ci\\04 common\\common_script.bat\" post static software")
                                            }
                                        }
                                    }
                                    updateGitlabCommitStatus name: 'SoftwareCompile', state: 'success'
                                }
                            }
                        }
                    }
                }
            }
            
            stage("Test") {
                parallel {
                    stage("Firmware") {
                        when { equals expected: Test_Firmware, actual: "Yes" }
                        stages {
                            stage('Firmware Test') {
                                agent { label "firmware_test" }
                                steps {
                                    updateGitlabCommitStatus name: 'FirmwareTest', state: 'running'
                                    // 下载代码
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

                                    // 固件测试
                                    dir("${WORKSPACE}/${GITLAB_GROUP}/${GITLAB_NAME}") {
                                        script {
                                            try {
                                                bat("call \"02 ci\\04 common\\common_script.bat\" test firmware")
                                            } catch (err) {
                                                echo "Caught error in firmware test: ${err}"
                                                currentBuild.result = 'FAILURE'
                                                updateGitlabCommitStatus name: 'FirmwareTest', state: 'failed'
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
                                    updateGitlabCommitStatus name: 'FirmwareTest', state: 'success'
                                }
                            }
                        }
                    }
                    stage("Software") {
                        when { equals expected: Test_Software, actual: "Yes" }
                        stages {
                            stage('Software Test') {
                                agent { label "software_test" }
                                steps {
                                    updateGitlabCommitStatus name: 'SoftwareTest', state: 'running'
                                    // 下载代码
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
                            
                                    // 软件测试
                                    dir("${WORKSPACE}/${GITLAB_GROUP}/${GITLAB_NAME}") {
                                        script {
                                            try {
                                                bat("call \"02 ci\\04 common\\common_script.bat\" test software")
                                            } catch (err) {
                                                echo "Caught error in software test: ${err}"
                                                currentBuild.result = 'FAILURE'
                                                updateGitlabCommitStatus name: 'SoftwareTest', state: 'failed'
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
                                    updateGitlabCommitStatus name: 'SoftwareTest', state: 'success'
                                }
                            }
                        }
                    }
                }
            }

            stage("Collection Reports") {
                steps {
                    updateGitlabCommitStatus name: 'CollectionReports', state: 'success'
                    dir("${WORKSPACE}/${GITLAB_GROUP}/${GITLAB_NAME}") {
                        sh "\"02 ci/04 common/control.sh\""
                    }
                }
            }
        }
        post {
            success {
                updateGitlabCommitStatus name: 'BuldComplete', state: 'success'
                emailext subject:"${GITLAB_NAME}, -Build #${TRIGGER_NUM} ${currentBuild.result}, commiter:${GITLAB_COMMITER}",
                         mimeType:'text/html',
                         body: '${FILE, path="summary.html"}', 
                         to: '${FILE,path="mail_list"} ${DEFAULT_RECIPIENTS}'
            }
            unstable {
                updateGitlabCommitStatus name: 'BuldComplete', state: 'failed'
                emailext subject:"${GITLAB_NAME}, -Build #${TRIGGER_NUM} ${currentBuild.result}, commiter:${GITLAB_COMMITER}",
                         mimeType:'text/html',
                         body: '${FILE, path="summary.html"}', 
                         to: '${FILE,path="mail_list"} ${DEFAULT_RECIPIENTS}'
            }
            failure {
                updateGitlabCommitStatus name: 'BuldComplete', state: 'failed'
                emailext subject:"${GITLAB_NAME}, -Build #${TRIGGER_NUM} ${currentBuild.result}, commiter:${GITLAB_COMMITER}",
                         mimeType:'text/html',
                         body: '${FILE, path="summary.html"}', 
                         to: '${FILE,path="mail_list"} ${DEFAULT_RECIPIENTS}'
            }
        }
    }

