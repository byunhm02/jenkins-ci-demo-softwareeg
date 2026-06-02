pipeline {
    agent any

    environment {
        SRC_DIR = 'src'
        TEST_DIR = 'test'
        LIB_DIR = 'lib'
        BUILD_DIR = 'build'
        TEST_REPORT_DIR = 'test-reports'
    }

    stages {
        stage('Checkout SCM') {
            steps {
                checkout scm
            }
        }

        stage('Prepare') {
            steps {
                sh 'mkdir -p ${BUILD_DIR}'
                sh 'mkdir -p ${TEST_REPORT_DIR}'
            }
        }

        stage('Build') {
            steps {
                sh 'javac -d ${BUILD_DIR} ${SRC_DIR}/*.java'
                echo 'Build 완료!'
            }
        }

        stage('Test') {
            steps {
                sh '''
                    javac -cp ${BUILD_DIR}:${LIB_DIR}/* -d ${BUILD_DIR} ${TEST_DIR}/*.java
                    java -jar ${LIB_DIR}/junit-platform-console-standalone-*.jar \
                        --class-path ${BUILD_DIR} \
                        --scan-class-path \
                        --reports-dir ${TEST_REPORT_DIR} \
                        --disable-ansi-colors \
                        --details=flat \
                        > test-output.txt 2>&1 || true
                '''
            }
        }
    }

    post {
        success {
            echo 'Build and test succeeded!'
            archiveArtifacts artifacts: 'test-output.txt', fingerprint: true
            junit 'test-reports/**/*.xml'
            sh '''
                echo "build Success! Time: $(date)" > build-result.txt
                echo "Build Number: ${BUILD_NUMBER}" >> build-result.txt
            '''
            archiveArtifacts artifacts: 'build-result.txt', fingerprint: true
            mail to: 'byunhm02@gmail.com',
                             subject: "Jenkins Build #${BUILD_NUMBER} Success",
                             body: "Build #${BUILD_NUMBER} succeeded!\nJob: ${JOB_NAME}"
        }
        failure {
            echo 'Build or test failed!'
            sh '''
                echo "Build Failed! Time: $(date)" > build-result.txt
                echo "Build Number: ${BUILD_NUMBER}" >> build-result.txt
            '''
            archiveArtifacts artifacts: 'build-result.txt', allowEmptyArchive: true
            mail to: 'byunhm02@gmail.com',
                             subject: "Jenkins Build #${BUILD_NUMBER} FAILED",
                             body: "Build #${BUILD_NUMBER} failed!\nJob: ${JOB_NAME}"
        }
    }
}