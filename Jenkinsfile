pipeline {
    agent { label 'linux x86' }
    stages {
        stage('Build') {
            steps {
                sh 'echo "Running on: $(hostname)"'
                sh 'echo "Architecture: $(uname -m)"'
            }
        }
    }
}
