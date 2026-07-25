pipeline {
    agent any

    environment {
        JMETER_HOME = "C:\\Users\\Youss\\Downloads\\apache-jmeter-5.6.3\\apache-jmeter-5.6.3\\bin"
        RESULTS_DIR = "logs"
        REPORT_DIR = "html\\report"
        TEST_PLAN = "api.jmx"
    }

    stages {

        stage('Checkout') {
            steps {
                echo "Checking out source code..."
                checkout scm
            }
        }

        stage('Prepare Workspace') {
            steps {
                bat """
                    if exist %RESULTS_DIR% rmdir /s /q %RESULTS_DIR%
                    if exist html rmdir /s /q html

                    mkdir %RESULTS_DIR%
                    mkdir html
                    mkdir %REPORT_DIR%
                """
            }
        }

        stage('Run JMeter Test') {
            steps {
                echo "Executing JMeter Performance Test..."

                bat """
                    "%JMETER_HOME%\\jmeter.bat" ^
                    -n ^
                    -t %TEST_PLAN% ^
                    -l %RESULTS_DIR%\\results.jtl ^
                    -e ^
                    -o %REPORT_DIR%
                """
            }
        }

        stage('Performance Gates') {
            steps {
                script {

                    // Analyse JMeter results
                    perfReport(
                        sourceDataFiles: 'logs/results.jtl',

                        // Performance Gates
                        errorFailedThreshold: 5,
                        errorUnstableThreshold: 2,

                        relativeFailedThresholdPositive: 20,
                        relativeUnstableThresholdPositive: 10,

                        modeOfThreshold: true
                    )
                }
            }
        }

        stage('Publish Reports') {
            steps {

                publishHTML(target: [
                    reportDir: 'html/report',
                    reportFiles: 'index.html',
                    reportName: 'JMeter HTML Report',
                    keepAll: true,
                    alwaysLinkToLastBuild: true,
                    allowMissing: false
                ])
            }
        }
    }

    post {

        success {
            echo "Performance Test Passed."
        }

        unstable {
            echo "Performance thresholds exceeded. Build marked UNSTABLE."
        }

        failure {
            echo "Performance test failed."
        }

        always {
            archiveArtifacts artifacts: 'logs/results.jtl, html/report/**'
        }

        cleanup {
            cleanWs()
        }
    }
}
