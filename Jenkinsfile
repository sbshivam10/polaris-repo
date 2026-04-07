pipeline {
    agent any

    tools {
        jdk 'openjdk-17'
        maven 'maven-3.9.11'
    }

    environment {
        POLARIS_SERVER_URL = 'https://poc.polaris.blackduck.com'
        POLARIS_APP_NAME   = 'Nirmal_SCM'
        POLARIS_PROJECT    = 'WebGoat'
        POLARIS_BRANCH     = 'Polaris-testing'

        // ✅ CORRECT build command for Java 17 + old Maven plugins
        COVERITY_BUILD_COMMAND = 'mvn clean compile -DskipTests -DskipITs -Dmaven.test.skip=true -Dgpg.skip=true'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Download Polaris Bridge CLI') {
            steps {
                sh '''
                  curl -fLsS -o bridge.zip \
                    https://repo.blackduck.com/bds-integrations-release/com/blackduck/integration/bridge/binaries/bridge-cli-bundle/latest/bridge-cli-bundle-linux64.zip
                  unzip -qo bridge.zip
                  chmod +x bridge-cli-bundle-linux64/bridge-cli
                '''
            }
        }

        stage('Run Polaris Scan') {
            environment {
                POLARIS_TOKEN = credentials('POLARIS_TOKEN')
            }
            steps {
                sh """
                  ./bridge-cli-bundle-linux64/bridge-cli --stage polaris \
                    polaris.serverUrl=${POLARIS_SERVER_URL} \
                    polaris.accessToken=${POLARIS_TOKEN} \
                    polaris.application.name=${POLARIS_APP_NAME} \
                    polaris.project.name=${POLARIS_PROJECT} \
                    polaris.branch.name=${POLARIS_BRANCH} \
                    polaris.assessment.types=SAST,SCA \
                    polaris.waitForScan=true \
                    polaris.reports.sarif.create=true \
                    coverity.build.command="${COVERITY_BUILD_COMMAND}"
                """
            }
        }
    }
}
