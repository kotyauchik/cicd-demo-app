pipeline {
    agent any

    parameters {
        string(name: 'REPO_URL', defaultValue: 'https://github.com/kotyauchik/cicd-demo-app.git', description: 'link na fork')
        string(name: 'GIT_BRANCH', defaultValue: 'master', description: 'branch for build')
        booleanParam(name: 'RUN_TESTS', defaultValue: true, description: 'start unit-tests')
    }

    stages {
        stage('Checkout') {
            steps {
                git url: params.REPO_URL, branch: params.GIT_BRANCH
                script {
                    def sha = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
                    echo "Branch: ${params.GIT_BRANCH}"
                    echo "Commit SHA: ${sha}"
                }
            }
        }

        stage('Validate') {
            steps {
                script {
                    def required = ['README.md', 'requirements.txt', 'src', 'tests']
                    def missing = required.findAll { !fileExists(it) }
                    if (missing) {
                        error "Missing required items: ${missing.join(', ')}"
                    }
                }
            }
        }

        stage('Test') {
            when { expression { params.RUN_TESTS } }
            steps {
                sh 'python3 -m unittest discover -s tests -v'
            }
        }

        stage('Build') {
            steps {
                script {
                    def sha = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
                    writeFile file: 'build-info.txt', text: "BUILD_NUMBER=${env.BUILD_NUMBER}\nCOMMIT_SHA=${sha}\n"
                    sh 'tar -czf app.tar.gz --exclude=.git README.md requirements.txt src tests build-info.txt'
                }
            }
        }

        stage('Archive') {
            steps {
                archiveArtifacts artifacts: 'app.tar.gz', fingerprint: true
            }
        }
    }

    post {
        success { echo "Build #${env.BUILD_NUMBER} succeeded." }
        failure { echo "Build #${env.BUILD_NUMBER} failed." }
    }
}
