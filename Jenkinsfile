pipeline {
    agent any

    stages {
        stage('Prepare Test Files') {
            steps {
                script {
                    // Build directory tree and dummy files
                    sh '''#!/bin/bash
                        mkdir -p first_pipeline/data/env1
                        mkdir -p first_pipeline/data/env2
                        mkdir -p first_pipeline/config

                        echo '{}' > first_pipeline/data/env1/test1.json
                        echo '{}' > first_pipeline/data/env1/test2.json
                        echo '{}' > first_pipeline/data/env2/test3.json
                        echo '{}' > first_pipeline/config/settings.json

                        echo "Created test file tree under first_pipeline/"
                    '''
                }
            }
        }

        stage('Upload Files to Nexus') {
            steps {
                script {
                    def uploadAllFilesToNexus = { String baseDir, String repoUrl, String repoPath, String credentialsId ->
                        def currentDate = new Date().format('dd-MM-yyyy')

                        if (!fileExists(baseDir)) {
                            error "Directory '${baseDir}' does not exist in workspace"
                        }

                        dir(baseDir) {
                            def output = sh(returnStdout: true, script: "find . -type f").trim()
                            if (!output) {
                                echo "No files found in '${baseDir}'"
                                return
                            }

                            def files = output.split('\n')

                            files.each { filePath ->
                                def relativePath = filePath.replaceFirst("^\\./", "")
                                def nexusUrl = "${repoUrl}/${repoPath}/${currentDate}/feature_test/${relativePath}"

                                echo "Uploading ${filePath} → ${nexusUrl}"

                                withCredentials([usernamePassword(credentialsId: credentialsId, passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
                                    sh """#!/bin/bash
                                        curl -s -o /dev/null -w "%{http_code}" -u "$USERNAME:$PASSWORD" \
                                             --upload-file '${filePath}' \
                                             '${nexusUrl}'
                                    """
                                }
                            }
                        }
                    }

                    uploadAllFilesToNexus(
                        'first_pipeline',
                        'http://localhost:8081/repository',
                        'iranRepo',
                        'nexus3'
                    )
                }
            }
        }
    }
}
