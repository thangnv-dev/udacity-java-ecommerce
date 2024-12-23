pipeline {

    agent {
        docker { image 'maven:3.6.3-openjdk-8-slim' }
    }

    environment {
        PROJECT_REPOSITORY_DIRECTORY = "eCommerce"
        APPLICATION_WAR_FILE = "target/ecommerce-0.0.1.war"
        APPLICATION_CONTEXT = "$WORKSPACE/$PROJECT_REPOSITORY_DIRECTORY"
    }

    options {
        skipDefaultCheckout()
        skipStagesAfterUnstable()
    }

    stages {

        stage ('SCM Checkout') {

            steps {
                checkout scm
            }

        }

        stage ('Build') {

            steps {

                script {
                    sh 'cd $APPLICATION_CONTEXT && mvn -B -DskipTests clean package'
                }

            }

        }

        stage ('Test') {

            steps {

                script {
                    sh 'cd $APPLICATION_CONTEXT && mvn clean test cobertura:cobertura -Dcobertura.report.format=xml'
                }

            }

            post {
                always {
                    junit '**/target/*-reports/TEST-*.xml'
                    step([$class: 'CoberturaPublisher', coberturaReportFile: 'eCommerce/target/site/cobertura/coverage.xml'])
                }
            }

        }

        stage ('Deploy') {

            steps {

                script {

                    def deploy_application = false

                    try {

                        echo 'Wait 5 minutes for the answer.'
                        timeout(time: 5, unit: 'MINUTES') {

                            deploy_application = input id: 'deploy_application',
                                message: 'Deploy the new version of the application?',
                                ok: 'Yes, deploy it.',
                                parameters: [
                                    [
                                        $class: 'BooleanParameterDefinition',
                                        defaultValue: true,
                                        successfulOnly: true,
                                        description: 'Please confirm you agree with this.',
                                        name: 'I agree with it.'
                                    ]
                                ]

                        }

                    } catch (error) {

                        def user = error.getCauses()[0].getUser()
                        if( 'SYSTEM' == user.toString() ) {
                            // The constant SYSTEM means that the build was cancelled by timeout.
                            echo 'No user interaction. Build timeout.'
                            currentBuild.result = 'ABORTED'
                            throw error
                        }
                        currentBuild.result = 'UNSTABLE'

                    }
                }   // Script

            }       // Steps

        } // Stage

    }   // Stages

}       // Pipeline
