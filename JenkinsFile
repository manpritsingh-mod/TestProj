@Library('My_UnifiedCI') _

pipeline{
    agent any

    environment{
        // These will be set dynamically based on YAML config file 
        PROJECT_LANGUAGE = ''
        BUILD_TOOL = ''
        RUN_UNIT_TESTS = ''
        RUN_LINT_TESTS = ''
        PROJECT_CONFIG = ''
    }

    stages{
        stage('Analysis / Processing YAML'){
            steps {
                script {
                    Logger.info("----STAGE 1: ANALYSIS / PROCESSING YAML----")
                    
                    // Read project configuration from YAML
                    def config = core_utils.readProjectConfig()
                    
                    // Converting the config map to a JSON string and assign it to the environment variable
                    // Convert map -> JSON string -> store in env var
                    env.PROJECT_CONFIG = writeJSON returnText: true, json: config
                    
                    Logger.info("-----YAML ANALYSIS RESULTS--------")
                    Logger.info("Project Language: ${config.project_language}")
                    Logger.info("Unit Tests: ${config.runUnitTests ? 'ENABLED' : 'DISABLED'}")
                    Logger.info("Lint Tests: ${config.runLintTests ? 'ENABLED' : 'DISABLED'}")
                    
                    if (config.tool_for_unit_testing) {
                        Logger.info("Unit Test Tools: ${config.tool_for_unit_testing}")
                    }
                    if (config.tool_for_lint_testing) {
                        Logger.info("Lint Test Tools: ${config.tool_for_lint_testing}")
                    }
                    Logger.info("YAML Analysis completed successfully")
                }
            }
        }
        
        stage('Prep Stage'){
            steps {
                script {
                    Logger.info("----- STAGE 2: PREP STAGE ----")
                    
                    def config = readJSON text: env.PROJECT_CONFIG
                    
                    // Setup global environment variables
                    core_utils.setupEnvironment()
                    
                    Logger.info("Global environment setup completed")
                }
            }
        }

        stage('Execute Project Template'){
            steps{
                script{
                    Logger.info("------STAGE 3: EXECUTE PROJECT TEMPLATE -------")
                
                    def config = readJSON text: env.PROJECT_CONFIG

                    // Calling appropriate template based on project language
                    Logger.info("Calling template for: ${config.project_language}")
                    
                    switch(config.project_language) {
                        case 'java-maven':
                            Logger.info("Executing Java Maven template")
                            javaMaven_template(config) // doubt in this
                            break
                        case 'java-gradle':
                            Logger.info("Executing Java Gradle template")
                            javaGradle_template(config) //doubt in this
                            break
                        case 'python':
                            Logger.info("Executing Python template")
                            python_template(config) // doubt in this
                            break
                        default:
                            Logger.error("Unsupported project language: ${config.project_language}")
                    }
                    Logger.info("Project template execution completed")
                }
            }
        }

        // Here Do we need to give any of the post notification message like success & failure ?? 
    }
}
