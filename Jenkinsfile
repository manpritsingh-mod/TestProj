@Library('My_UnifiedCI') _

pipeline {
    agent any

    tools {
        maven 'Maven 3.8.1' 
        gradle 'Gradle 7.5'
    }

    environment {
        // These will be set dynamically based on YAML config file 
        PROJECT_LANGUAGE = ''
        BUILD_TOOL = ''
        RUN_UNIT_TESTS = ''
        RUN_LINT_TESTS = ''
    }

    stages {
        stage('Setup and Execution') {
            steps {
                script {
                    echo "---- STAGE: SETUP AND EXECUTION ----"

                    // Read project configuration from YAML
                    def config = core_utils.readProjectConfig()
                    echo "Config map content: ${config}"

                    // Check if the configuration map is not empty or null
                    if (config && !config.isEmpty()) { // config is containing the LinkedHashMap so I am usnig isEmpth()
                        // Setup global environment
                        core_utils.setupEnvironment()
                        echo "Global environment setup completed"
                    } else {
                        error("PROJECT_CONFIG is empty or missing")
                    }

                    // Call appropriate template based on the project language
                    echo "Calling template for: ${config.project_language}"

                    switch (config.project_language) {
                        case 'java-maven':
                            echo "Executing Java Maven template"
                            javaMaven_template(config)
                            break
                        case 'java-gradle':
                            echo "Executing Java Gradle template"
                            javaGradle_template(config)
                            break
                        case 'python':
                            echo "Executing Python template"
                            python_template(config)
                            break
                        default:
                            echo "Unsupported project language: ${config.project_language}"
                    }

                    echo "Project template execution completed"
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
