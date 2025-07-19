@Library('My_UnifiedCI') _

pipeline {
    agent any
    
    tools {
        maven 'Maven 3.8.1'
        gradle 'Gradle 7.5'
        allure 'Allure-2.34.1'
    }
    
    environment {
        // Basic environment variables
        PROJECT_LANGUAGE = ''
        BUILD_TOOL = ''
        RUN_UNIT_TESTS = ''
        RUN_LINT_TESTS = ''
        
        // Stage status tracking (simple strings, not JSON)
        LINT_STATUS = ''
        TEST_STATUS = ''
    }
    
    stages {
        stage('Setup and Execution') {
            steps {
                script {
                    logger.info("---- STAGE: SETUP AND EXECUTION ----")
                    
                    // Read project configuration from YAML
                    def config = core_utils.readProjectConfig()
                    logger.info("Config map content: ${config}")
                    
                    if (config && !config.isEmpty()) {
                        // Setup global environment
                        core_utils.setupEnvironment()
                        logger.info("Global environment setup completed")
                        
                        try {
                            // Call appropriate template based on the project language
                            logger.info("Calling template for: ${config.project_language}")
                            
                            switch (config.project_language) {
                                case 'java-maven':
                                    logger.info("Executing Java Maven template")
                                    javaMaven_template(config)
                                    break
                                case 'java-gradle':
                                    logger.info("Executing Java Gradle template")
                                    javaGradle_template(config)
                                    break
                                case 'python':
                                    logger.info("Executing Python template")
                                    python_template(config)
                                    break
                                default:
                                    error("Unsupported project language: ${config.project_language}")
                            }
                            
                            logger.info("Project template execution completed")
                            
                        } catch (Exception e) {
                            logger.error("Pipeline execution failed: ${e.getMessage()}")
                            
                            // Set build result based on quality gates
                            if (config.quality_gates?.lint?.fail_on_error == false || 
                                config.quality_gates?.unit_test?.fail_on_error == false) {
                                currentBuild.result = 'UNSTABLE'
                                logger.warning("Build marked as UNSTABLE due to quality gate settings")
                            } else {
                                currentBuild.result = 'FAILURE'
                                logger.error("Build marked as FAILED")
                            }
                            
                            logger.warning("Continuing to post actions for notification handling")
                        }
                        
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
                logger.info("=== POST ACTIONS STARTED ===")
                
                try {
                    // Read config directly (don't rely on stored env variables)
                    def config = core_utils.readProjectConfig()
                    
                    // Create stage results based on environment variables
                    def stageResults = [:]
                    def buildStatus = currentBuild.result ?: 'SUCCESS'
                    
                    // Add stage information if available
                    if (env.LINT_STATUS && env.LINT_STATUS.trim()) {
                        stageResults['Lint'] = env.LINT_STATUS
                        logger.info("Lint Status: ${env.LINT_STATUS}")
                    }
                    if (env.TEST_STATUS && env.TEST_STATUS.trim()) {
                        stageResults['Unit Tests'] = env.TEST_STATUS
                        logger.info("Test Status: ${env.TEST_STATUS}")
                    }
                    
                    logger.info("Final Build Status: ${buildStatus}")
                    logger.info("Stage Results: ${stageResults}")
                    
                    // Generate and send reports if config is available
                    if (config && !config.isEmpty()) {
                        logger.info("Generating comprehensive reports...")
                        sendReport.generateAndSendReports(config, stageResults)
                        
                        logger.info("Sending build notification...")
                        notify.notifyBuildStatus(buildStatus, config)
                    } else {
                        logger.warning("No config available, sending basic notification")
                        def basicConfig = [
                            notifications: [
                                email: [recipients: ["smanprit022@gmail.com"]]
                            ]
                        ]
                        notify.notifyBuildStatus(buildStatus, basicConfig)
                    }
                    
                } catch (Exception e) {
                    logger.error("Failed in post actions: ${e.getMessage()}")
                    
                    // Simple fallback notification
                    try {
                        logger.info("Attempting fallback notification...")
                        def fallbackConfig = [
                            notifications: [
                                email: [recipients: ["smanprit022@gmail.com"]]
                            ]
                        ]
                        notify.notifyBuildStatus('FAILED', fallbackConfig)
                        logger.info("Fallback notification sent successfully")
                    } catch (Exception fallbackError) {
                        logger.error("Fallback notification also failed: ${fallbackError.getMessage()}")
                    }
                }
                
                logger.info("=== POST ACTIONS COMPLETED ===")
            }
        }
        
        success {
            script {
                logger.info("BUILD SUCCESSFUL!")
                logger.info("All stages completed without critical failures")
            }
        }
        
        failure {
            script {
                logger.error("BUILD FAILED!")
                logger.error("One or more critical stages failed")
            }
        }
        
        unstable {
            script {
                logger.warning("BUILD UNSTABLE!")
                logger.warning("Build completed but with warnings or non-critical failures")
                logger.warning("This typically means lint or unit tests failed but were configured as non-critical")
            }
        }
        
        aborted {
            script {
                logger.warning("BUILD ABORTED!")
                logger.warning("Build was manually cancelled or timed out")
                
                // Send aborted notification
                try {
                    def config = core_utils.readProjectConfig()
                    if (config && !config.isEmpty()) {
                        notify.notifyBuildStatus('ABORTED', config)
                    } else {
                        def basicConfig = [
                            notifications: [
                                email: [recipients: ["smanprit022@gmail.com"]]
                            ]
                        ]
                        notify.notifyBuildStatus('ABORTED', basicConfig)
                    }
                } catch (Exception e) {
                    logger.error("Failed to send abort notification: ${e.getMessage()}")
                }
            }
        }
        
        cleanup {
            script {
                logger.info("CLEANUP PHASE")
                logger.info("Build #${env.BUILD_NUMBER} cleanup completed")
            }
        }
    }
}
