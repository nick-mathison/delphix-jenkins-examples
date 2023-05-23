pipeline {
    agent any

      parameters{
        string(name: 'vdb_name', defaultValue: '', description: "Enter an option VDB Name.")
        string(name: 'bookmark_id', defaultValue: '', required: true)
      }
    
    environment {
        def vdb_name = "jkns${env.BUILD_NUMBER}";
    }

    stages {
        stage ('Provision 1 VDB') {
            steps {
                script {
                    if (input.vdb_name == "") {
                        vdb_name = input.vdb_name;
                    }
                    withCredentials([string(credentialsId: 'DCT_API_KEY', variable: 'KEY')]) {
                        delphixProvision name=vdb_name bookmark="${input.bookmark_id}" dct_api_key="${KEY}"
                    }
                }
            }
        }

        stage ('Run Selenium Tests') {
            steps{
                echo ('Selenium Tests...')
                sleep (5)
            }
        }
        stage ('Send UI Test Results Via Email') {
            steps{
                script {
                    echo ('Sending UI Test Results via email...')
                    def props = readProperties file: './delphix.properties'
                    // Jenkins Plugin: https://plugins.jenkins.io/email-ext/
                    emailext body: """
                        You test results are 95% passing.
                        
                        ~ VDB Information ~
                        Name: ${props[vdb_id]}
                        IP: ${props[vdb_ip]}
                        Port: ${props[vdb_port]}
                        User: ${props[vdb_user]}
                        Password: ****
                    """,
                        subject: 'Test Subject',
                        to: 'test@example.com'
                }
            }
        }

        stage ('Destroy Full Application') {
            steps {
                script {
                    // Option #1 with ID
                    def props = readProperties file: './delphix.properties'
                    withCredentials([string(credentialsId: 'DCT_API_KEY', variable: 'KEY')]) {
                        delphixDestroy id="${props[vdb_id]}" dct_api_key=$KEY
                    }
                    // Option #2 with checkbox
                    withCredentials([string(credentialsId: 'DCT_API_KEY', variable: 'KEY')]) {
                        delphixDestroy use_properties_file=true dct_api_key=$KEY
                    }
                    // Option #3 with Name
                    // We might need additional internal logic to transform the Name to an ID which the DCT API understands
                    withCredentials([string(credentialsId: 'DCT_API_KEY', variable: 'KEY')]) {
                        delphixDestroy name=vdb_name dct_api_key=$KEY
                    }
                }
            }
        }
    }
}
