@Library('My_UnifiedCI') _

pipeline {
    agent any
    
    tools {
        maven 'Maven 3.8.1'
        gradle 'Gradle 7.5'
        allure 'Allure-2.34.1'
    }
    
    environment {
        PROJECT_LANGUAGE = ''
        BUILD_TOOL = ''
        RUN_UNIT_TESTS = ''
        RUN_LINT_TESTS = ''
    }
    
    stages {
        stage('Setup and Execution') {
            steps {
                script {
                    logger.info("---- STAGE: SETUP AND EXECUTION ----")
                    
                    def config = core_utils.readProjectConfig()
                    logger.info("Config map content: ${config}")
                    
                    if (config && !config.isEmpty()) {
                        core_utils.setupEnvironment()
                        logger.info("Global environment setup completed")
                        
                        // Call template
                        logger.info("Calling template for: ${config.project_language}")
                        switch (config.project_language) {
                            case 'java-maven':
                                javaMaven_template(config)
                                break
                            case 'java-gradle':
                                javaGradle_template(config)
                                break
                            case 'python':
                                python_template(config)
                                break
                            default:
                                error("Unsupported project language: ${config.project_language}")
                        }
                        
                        logger.info("Project template execution completed")
                    } else {
                        error("PROJECT_CONFIG is empty or missing")
                    }
                }
            }
        }
    }
    post {
        always {
            script {
                logger.info("=== SENDING NOTIFICATIONS ===")
                def buildStatus = currentBuild.result ?: 'SUCCESS'
                def config = [
                    notifications: [
                        email: [recipients: ["smanprit022@gmail.com"]]
                    ]
                ]
                
                notify.notifyBuildStatus(buildStatus, config)
                logger.info("Notification sent successfully")
            }
        }
        
        success {
            script {
                logger.info("BUILD SUCCESSFUL!")
            }
        }
        
        failure {
            script {
                logger.error("BUILD FAILED!")
            }
        }
        
        unstable {
            script {
                logger.warning("BUILD UNSTABLE!")
            }
        }
    }
}
