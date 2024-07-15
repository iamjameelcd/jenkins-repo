pipeline {
    agent any

    parameters {
        choice(name: 'AnzoEnvironment', choices: ['stage'], description: 'Environment to use')
        string(name: 'MigrationPackageName', defaultValue: '', description: 'Name of the migration package')
        string(name: 'MigrationPackageURI', defaultValue: '', description: 'URI of the migration package')
        choice(name: 'MigrationPackagePromotion', choices: ['StageToQA', 'StageToProd'], description: 'Promotion stage')
    }

    environment {
        ANZO_DEV_HOST = 'your-anzo-dev-ip-or-hostname'
        ANZO_QA_HOST = 'your-anzo-qa-ip-or-hostname'
        EC2_USER = 'ec2-user'
    }

    stages {
        stage('Verify Tools') {
            steps {
                script {
                    def missingToolsAnzoDev = [:]
                    def missingToolsAnzoQa = [:]

                    // Fetch credentials for sysadmin user from Jenkins credentials
                    withCredentials([usernamePassword(credentialsId: '8b6cd893-8251-45d5-ba40-cfd71b3ac003', passwordVariable: 'SYSADMIN_PASSWORD', usernameVariable: 'SYSADMIN_USER')]) {
                        
                        // Verify tools on anzo-dev
                        sshagent(credentials: ['12235c6a-9627-4aa8-b1d6-ffbec7e7a9df']) {
                            def anzoFound = sh(script: """
                                ssh -o StrictHostKeyChecking=yes ${EC2_USER}@${ANZO_DEV_HOST} '
                                    anzoFound=false
                                    jqFound=false
                                    curlFound=false

                                    if [ -f /opt/anzo/Client/anzo ]; then
                                        anzoFound=true
                                    fi
                                    if command -v jq > /dev/null; then
                                        jqFound=true
                                    fi
                                    if command -v curl > /dev/null; then
                                        curlFound=true
                                    fi

                                    if ! $anzoFound || ! $jqFound || ! $curlFound; then
                                        echo "Missing tools: "
                                        if ! $anzoFound; then
                                            echo "  - anzo"
                                        fi
                                        if ! $jqFound; then
                                            echo "  - jq"
                                        fi
                                        if ! $curlFound; then
                                            echo "  - curl"
                                        fi
                                        exit 1
                                    fi
                                '
                            """, returnStatus: true)

                            if (anzoFound != 0) {
                                missingToolsAnzoDev['anzo'] = true
                            }
                            if (jqFound != 0) {
                                missingToolsAnzoDev['jq'] = true
                            }
                            if (curlFound != 0) {
                                missingToolsAnzoDev['curl'] = true
                            }
                        }

                        // Verify tools on anzo-qa
                        sshagent(credentials: ['2658e5ae-f74b-49cb-8b1f-bcfab3b9b608']) {
                            def anzoFound = sh(script: """
                                ssh -o StrictHostKeyChecking=yes ${EC2_USER}@${ANZO_QA_HOST} '
                                    anzoFound=false
                                    jqFound=false
                                    curlFound=false

                                    if [ -f /opt/anzo/Client/anzo ]; then
                                        anzoFound=true
                                    fi
                                    if command -v jq > /dev/null; then
                                        jqFound=true
                                    fi
                                    if command -v curl > /dev/null; then
                                        curlFound=true
                                    fi

                                    if ! $anzoFound || ! $jqFound || ! $curlFound; then
                                        echo "Missing tools: "
                                        if ! $anzoFound; then
                                            echo "  - anzo"
                                        fi
                                        if ! $jqFound; then
                                            echo "  - jq"
                                        fi
                                        if ! $curlFound; then
                                            echo "  - curl"
                                        fi
                                        exit 1
                                    fi
                                '
                            """, returnStatus: true)

                            if (anzoFound != 0) {
                                missingToolsAnzoQa['anzo'] = true
                            }
                            if (jqFound != 0) {
                                missingToolsAnzoQa['jq'] = true
                            }
                            if (curlFound != 0) {
                                missingToolsAnzoQa['curl'] = true
                            }
                        }
                    }

                    // Fail the pipeline if any tools are missing
                    if (missingToolsAnzoDev || missingToolsAnzoQa) {
                        def missingToolsMessage = []

                        if (missingToolsAnzoDev) {
                            missingToolsMessage << "Missing tools on anzo-dev: ${missingToolsAnzoDev.keySet().join(', ')}"
                        }
                        if (missingToolsAnzoQa) {
                            missingToolsMessage << "Missing tools on anzo-qa: ${missingToolsAnzoQa.keySet().join(', ')}"
                        }

                        error(missingToolsMessage.join('\n'))
                    } else {
                        echo "All required tools (anzo, jq, curl) are installed on both anzo-dev and anzo-qa"
                    }
                }
            }
        }

        stage('List Migration Package') {
            steps {
                withCredentials([usernamePassword(credentialsId: '8b6cd893-8251-45d5-ba40-cfd71b3ac003', passwordVariable: 'SYSADMIN_PASSWORD', usernameVariable: 'SYSADMIN_USER')]) {
                    script {
                        if (!params.MigrationPackageName || !params.MigrationPackageURI) {
                            error("MigrationPackageName or MigrationPackageURI parameter is missing.")
                        }

                        // List migration package on anzo-dev
                        def anzoDevCurlCommand = "ssh -o StrictHostKeyChecking=yes ${EC2_USER}@${ANZO_DEV_HOST} \"curl -k -X GET ''https://dev-anzo.dataplatform.rtx.com/api/migration/http://cambridgesemantics.com/MigrationPackage/ebd7b0339df349c9b4e68a078785a0c4' -H 'accept: application/json' -u ${SYSADMIN_USER}:${SYSADMIN_PASSWORD} | jq\""
                        echo "Executing command on anzo-dev: ${anzoDevCurlCommand.replaceAll(SYSADMIN_PASSWORD, '****')}"
                        def anzoDevResponse = sh(script: anzoDevCurlCommand, returnStdout: true).trim()
                        echo "Migration Package Details on anzo-dev: ${anzoDevResponse}"

                        // List migration package on anzo-qa
                        def anzoQaCurlCommand = "ssh -o StrictHostKeyChecking=yes ${EC2_USER}@${ANZO_QA_HOST} \"curl -k -X GET ''https://dev-anzo.dataplatform.rtx.com/api/migration/http://cambridgesemantics.com/MigrationPackage/ebd7b0339df349c9b4e68a078785a0c4' -H 'accept: application/json' -u ${SYSADMIN_USER}:${SYSADMIN_PASSWORD} | jq\""
                        echo "Executing command on anzo-qa: ${anzoQaCurlCommand.replaceAll(SYSADMIN_PASSWORD, '****')}"
                        def anzoQaResponse = sh(script: anzoQaCurlCommand, returnStdout: true).trim()
                        echo "Migration Package Details on anzo-qa: ${anzoQaResponse}"
                    }
                }
            }
        }
    }
}
