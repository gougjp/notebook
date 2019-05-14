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

