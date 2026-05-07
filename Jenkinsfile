// ============================================
// Jenkins CI/CD Pipeline
// UI + API Hybrid Automation Framework
// ============================================

pipeline {

    agent any

    parameters {

        choice(
            name: 'BROWSER',
            choices: ['chrome', 'firefox', 'edge'],
            description: 'Browser for UI tests'
        )

        choice(
            name: 'ENV',
            choices: ['dev', 'staging', 'production'],
            description: 'Target environment'
        )

        booleanParam(
            name: 'HEADLESS',
            defaultValue: true,
            description: 'Run browser in headless mode'
        )

        string(
            name: 'PARALLEL_WORKERS',
            defaultValue: '2',
            description: 'Number of parallel workers'
        )
    }

    environment {

        TEST_ENV = "${params.ENV}"
        BROWSER = "${params.BROWSER}"
        HEADLESS = "${params.HEADLESS}"
        PYTHONPATH = "${WORKSPACE}"
    }

    stages {

        // ============================================
        // Checkout Source Code
        // ============================================

        stage('Checkout') {

            steps {

                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/RAKESH-564/Capstone-Project_QA.git'
                    ]]
                ])

                echo 'Code checked out successfully from GitHub repository'
            }
        }

        // ============================================
        // Setup Python Environment
        // ============================================

        stage('Setup Environment') {

            steps {

                script {

                    if (isUnix()) {

                        sh '''
                            python3 -m venv venv
                            . venv/bin/activate
                            pip install --upgrade pip
                            pip install -r requirements.txt
                        '''

                    } else {

                        bat '''
                            python -m venv venv
                            call venv\\Scripts\\activate
                            pip install --upgrade pip
                            pip install -r requirements.txt
                        '''
                    }
                }
            }
        }

        // ============================================
        // API Health Check
        // ============================================

        stage('API Health Check') {

            steps {

                script {

                    if (isUnix()) {

                        sh '''
                            . venv/bin/activate
                            python -c "import requests; r=requests.get('https://practice.expandtesting.com/notes/api/health-check'); print(f'API Status: {r.status_code}')"
                        '''

                    } else {

                        bat '''
                            call venv\\Scripts\\activate
                            python -c "import requests; r=requests.get('https://practice.expandtesting.com/notes/api/health-check'); print(f'API Status: {r.status_code}')"
                        '''
                    }
                }
            }
        }

        // ============================================
        // Run Automation Tests
        // ============================================

        stage('Run Tests') {

            steps {

                script {

                    def cmd = """
                    pytest tests/ -v -s \
                    -n ${params.PARALLEL_WORKERS} \
                    --dist=loadfile \
                    --junitxml=reports/results.xml \
                    --alluredir=reports/allure-results \
                    --html=reports/report.html \
                    --self-contained-html \
                    --reruns=2 \
                    --reruns-delay=2
                    """

                    runTests(cmd)
                }
            }
        }

        // ============================================
        // Generate Allure HTML Report
        // ============================================

        stage('Generate Allure Report') {

            steps {

                script {

                    if (isUnix()) {

                        sh '''
                            allure generate reports/allure-results \
                            -o reports/allure-report \
                            --clean
                        '''

                    } else {

                        bat '''
                            allure generate reports\\allure-results ^
                            -o reports\\allure-report ^
                            --clean
                        '''
                    }
                }
            }
        }

        // ============================================
        // Publish HTML Reports
        // ============================================

        stage('Publish Reports') {

            steps {

                publishHTML([
                    allowMissing: true,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: 'reports',
                    reportFiles: 'report.html',
                    reportName: 'Pytest HTML Report'
                ])

                publishHTML([
                    allowMissing: true,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: 'reports/allure-report',
                    reportFiles: 'index.html',
                    reportName: 'Allure HTML Report'
                ])
            }
        }
    }

    // ============================================
    // Post Actions
    // ============================================

    post {

        always {

            echo 'Archiving test artifacts...'

            archiveArtifacts(
                artifacts: 'reports/**/*',
                allowEmptyArchive: true
            )

            junit(
                testResults: 'reports/results.xml',
                allowEmptyResults: true
            )
        }

        success {

            echo '✅ All tests PASSED!'
        }

        failure {

            echo '❌ Some tests FAILED. Check reports for details.'
        }

        cleanup {

            cleanWs()
        }
    }
}

// ============================================
// Helper Function
// ============================================

def runTests(String command) {

    if (isUnix()) {

        sh """
            . venv/bin/activate
            ${command}
        """

    } else {

        bat """
            call venv\\Scripts\\activate
            ${command}
        """
    }
}
