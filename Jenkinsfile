pipeline {
    agent any

    environment {
        // Root JMeter directory (NOT the bin folder)
        JMETER_HOME = "C:\\Users\\Youss\\Downloads\\apache-jmeter-5.6.3\\apache-jmeter-5.6.3"
    }

    stages {

        stage('Prepare Workspace') {
            steps {
                echo "Preparing workspace..."

                bat '''
                    if exist logs rmdir /s /q logs
                    if exist html rmdir /s /q html

                    mkdir logs
                    mkdir html
                    mkdir html\\report
                '''
            }
        }

        stage('Debug Environment') {
            steps {
                echo "Checking environment..."

                bat '''
                    echo ================================
                    echo Current Directory:
                    cd

                    echo.
                    echo Workspace Contents:
                    dir

                    echo.
                    echo Java Version:
                    java -version

                    echo.
                    echo JMETER_HOME:
                    echo %JMETER_HOME%

                    echo.
                    echo JMeter Folder:
                    dir "%JMETER_HOME%"

                    echo.
                    echo JMeter Bin:
                    dir "%JMETER_HOME%\\bin"

                    echo.
                    echo Checking api.jmx...
                    if exist api.jmx (
                        echo api.jmx FOUND
                    ) else (
                        echo api.jmx NOT FOUND
                        exit /b 1
                    )

                    echo ================================
                '''
            }
        }

        stage('Run JMeter Test') {
            steps {
                echo "Running JMeter..."

                bat '''
                    "%JMETER_HOME%\\bin\\jmeter.bat" -n ^
                    -t api.jmx ^
                    -l logs\\results.jtl ^
                    -e ^
                    -o html\\report
                '''
            }
        }

        stage('Verify Results') {
            steps {
                bat '''
                    if exist logs\\results.jtl (
                        echo Results file generated successfully.
                    ) else (
                        echo ERROR: results.jtl not found!
                        exit /b 1
                    )
                '''
            }
        }

        stage('Performance Gates') {
            steps {
                perfReport(
                    sourceDataFiles: 'logs/results.jtl',
                    errorFailedThreshold: 5,
                    errorUnstableThreshold: 2
                )
            }
        }

        stage('Publish HTML Report') {
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
            echo "SUCCESS: Performance test completed successfully."
        }

        unstable {
            echo "UNSTABLE: Performance thresholds exceeded."
        }

        failure {
            echo "FAILURE: Pipeline failed."
        }

        always {
            archiveArtifacts artifacts: 'logs/results.jtl, html/report/**', fingerprint: true
        }
    }
}
