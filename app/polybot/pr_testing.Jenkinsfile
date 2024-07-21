pipeline {
    agent any


    stages {
        stage("Unittests") {
            steps {
             sh 'echo "unittests"'
            }
        }

        stage("lint") {
            steps {
                sh 'echo "lint"'
            }
        }

        stage("Functional test"){
            steps {
                sh 'echo "functional test"'
            }
        }
    }
    /*
    to ignore pylint:
    [MESSAGES CONTROL]
disable=missing-module-docstring, import-error, missing-function-docstring
    */
}
