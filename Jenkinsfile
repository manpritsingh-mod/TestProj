@Library('My_UnifiedCI') _

pipeline {
    agent any

    environment {
        // These will be set dynamically based on YAML config file 
        PROJECT_LANGUAGE = ''
        BUILD_TOOL = ''
        RUN_UNIT_TESTS = ''
        RUN_LINT_TESTS = ''
        PROJECT_CONFIG = ''
    }

    stages {
        stage('Analysis / Processing YAML') {
            steps {
                script {
                    // Logger.info("----STAGE 1: ANALYSIS / PROCESSING YAML----")
                    echo "----STAGE 1: ANALYSIS / PROCESSING YAML----"
                    

                    // Read project configuration from YAML
                    def config = core_utils.readProjectConfig()
                    // Logger.info("Config map content: ${config}")
                    echo "Config map content: ${config}"

                    // Convert map to JSON string and store it in env var
                    def jsonConfig = writeJSON returnText: true, json: config
                    writeFile file: 'project_config.json', text: jsonConfig
                    echo "Config JSON saved to project_config.json"

                    echo "printing the envirnment variable: ${jsonConfig}"

                    // env.PROJECT_CONFIG = readJSON text: jsonConfig

                    // echo "Now what value is been shown ${env.PROJECT_CONFIG}" 

                    // Logger.info("-----YAML ANALYSIS RESULTS--------")
                    // Logger.info("Project Language: ${config.project_language}")
                    // Logger.info("Unit Tests: ${config.runUnitTests ? 'ENABLED' : 'DISABLED'}")
                    // Logger.info("Lint Tests: ${config.runLintTests ? 'ENABLED' : 'DISABLED'}")

                    echo "-----YAML ANALYSIS RESULTS--------"
                    echo "Project Language: ${config.project_language}"
                    echo "Unit Tests: ${config.runUnitTests ? 'ENABLED' : 'DISABLED'}"
                    echo "Lint Tests: ${config.runLintTests ? 'ENABLED' : 'DISABLED'}"

                    if (config.tool_for_unit_testing) {
                        // Logger.info("Unit Test Tools: ${config.tool_for_unit_testing}")
                        echo "Unit Test Tools: ${config.tool_for_unit_testing}"
                    }
                    if (config.tool_for_lint_testing) {
                        // Logger.info("Lint Test Tools: ${config.tool_for_lint_testing}")
                        echo "Lint Test Tools: ${config.tool_for_lint_testing}"
                        
                    }
                    // Logger.info("YAML Analysis completed successfully")
                    echo "YAML Analysis completed successfully"
                }
            }
        }

        stage('Prep Stage') {
            steps {
                script {
                    // Logger.info("----- STAGE 2: PREP STAGE ----")
                    echo "----- STAGE 2: PREP STAGE ----"
                    if (env.PROJECT_CONFIG?.trim()) {
                        // def config = readJSON text: env.PROJECT_CONFIG
                        def jsonText = readFile('project_config.json')
                        def config = readJSON text: jsonText
                        echo "Loaded config from JSON file: ${config}"

                        core_utils.setupEnvironment(config) 
                        // Logger.info("Global environment setup completed")
                        echo "Global environment setup completed"
                    } else {
                        // error("PROJECT_CONFIG is empty or missing")
                        echo "PROJECT_CONFIG is empty or missing"
                    }
                }
            }
        }

        stage('Execute Project Template') {
            steps {
                script {
                    Logger.info("------STAGE 3: EXECUTE PROJECT TEMPLATE -------")

                    // Ensure PROJECT_CONFIG is correctly parsed
                    def config = readJSON text: env.PROJECT_CONFIG

                    // Call appropriate template based on the project language
                    Logger.info("Calling template for: ${config.project_language}")

                    switch (config.project_language) {
                        case 'java-maven':
                            Logger.info("Executing Java Maven template")
                            // Ensure your template function is available
                            javaMaven_template(config) 
                            break
                        case 'java-gradle':
                            Logger.info("Executing Java Gradle template")
                            // Ensure your template function is available
                            javaGradle_template(config) 
                            break
                        case 'python':
                            Logger.info("Executing Python template")
                            // Ensure your template function is available
                            python_template(config) 
                            break
                        default:
                            Logger.error("Unsupported project language: ${config.project_language}")
                            error("Unsupported project language: ${config.project_language}")
                    }
                    Logger.info("Project template execution completed")
                }
            }
        }
    }

    // post {
    //     success {
    //         // Notify success (Slack, Email)
    //         Logger.info("Pipeline completed successfully!")
    //     }
    //     failure {
    //         // Notify failure (Slack, Email)
    //         Logger.error("Pipeline failed!")
    //     }
    // }
}
