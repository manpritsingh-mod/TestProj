@Library('My_UnifiedCI') _

pipeline {
    agent any
    
    tools {
        maven 'Maven 3.8.1'
        gradle 'Gradle 7.5'
        allure 'Allure-2.34.1'
    }
    
    environment {
        // These will be set dynamically based on YAML config file
        PROJECT_LANGUAGE = ''
        BUILD_TOOL = ''
        RUN_UNIT_TESTS = ''
        RUN_LINT_TESTS = ''
        
        // NEW: Stage tracking variables
        STAGE_RESULTS = ''
        BUILD_STATUS = 'SUCCESS'
        PROJECT_CONFIG = ''
    }
    
    stages {
        stage('Setup and Execution') {
            steps {
                script {
                    logger.info("---- STAGE: SETUP AND EXECUTION ----")
                    
                    // Read project configuration from YAML
                    def config = core_utils.readProjectConfig()
                    logger.info("Config map content: ${config}")
                    
                    // NEW: Initialize stage results tracking
                    def stageResults = [:]
                    
                    // Check if the configuration map is not empty or null
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
                                    stageResults = javaMaven_template(config)
                                    break
                                case 'java-gradle':
                                    logger.info("Executing Java Gradle template")
                                    stageResults = javaGradle_template(config)
                                    break
                                case 'python':
                                    logger.info("Executing Python template")
                                    stageResults = python_template(config)
                                    break
                                default:
                                    error("Unsupported project language: ${config.project_language}")
                            }
                            
                            logger.info("Project template execution completed")
                            
                            // NEW: Determine overall build status based on stage results
                            env.BUILD_STATUS = notify.determineBuildStatus(stageResults)
                            logger.info("Determined build status: ${env.BUILD_STATUS}")
                            
                        } catch (Exception e) {
                            logger.error("Pipeline execution failed: ${e.getMessage()}")
                            
                            // NEW: Handle failures - set status and continue to post actions
                            env.BUILD_STATUS = 'FAILED'
                            stageResults['Error'] = 'FAILED'
                            
                            // Set Jenkins build result for post actions
                            if (config.quality_gates?.lint?.fail_on_error == false || 
                                config.quality_gates?.unit_test?.fail_on_error == false) {
                                currentBuild.result = 'UNSTABLE'
                                env.BUILD_STATUS = 'UNSTABLE'
                            } else {
                                currentBuild.result = 'FAILURE'
                            }
                            
                            // Don't re-throw - let post actions handle notifications
                            logger.warning("Continuing to post actions for notification handling")
                        } finally {
                            // NEW: Store results for post actions
                            env.STAGE_RESULTS = writeJSON returnText: true, json: stageResults
                            env.PROJECT_CONFIG = writeJSON returnText: true, json: config
                        }
                        
                    } else {
                        error("PROJECT_CONFIG is empty or missing")
                    }
                }
            }
        }
    }
    
    // NEW: Post actions for comprehensive notification handling
    post {
        always {
            script {
                logger.info("=== POST ACTIONS STARTED ===")
                
                try {
                    // Read stored configuration and results
                    def config = readJSON text: env.PROJECT_CONFIG
                    def stageResults = readJSON text: env.STAGE_RESULTS
                    def buildStatus = env.BUILD_STATUS
                    
                    logger.info("Final Build Status: ${buildStatus}")
                    logger.info("Stage Results: ${stageResults}")
                    
                    // Send basic notification only
                    notify.notifyBuildStatus(buildStatus, config)
                    
                } catch (Exception e) {
                    logger.error("Failed to send notifications: ${e.getMessage()}")
                    // Fallback notification
                    try {
                        def fallbackConfig = [
                            notifications: [
                                email: [recipients: ['smanprit022@gmail.com']],
                                slack: [channel: '#builds']
                            ]
                        ]
                        notify.notifyBuildStatus('FAILED', fallbackConfig)
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
                    def config = readJSON text: env.PROJECT_CONFIG
                    notify.notifyBuildStatus('ABORTED', config)
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
