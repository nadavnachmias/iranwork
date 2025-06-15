def uploadAllFilesToNexus(String baseDir, String repoUrl, String repoPath, String credentialsId) {
    def currentDate = new Date().format('dd-MM-yyyy')

    dir(baseDir) {
        // Find all files under the baseDir recursively
        def files = sh(returnStdout: true, script: "find . -type f").trim().split('\n')

        files.each { filePath ->
            def relativePath = filePath.replaceFirst("^\\./", "")
            def nexusUrl = "${repoUrl}/${repoPath}/${currentDate}/feature_test/${relativePath}"

            withCredentials([usernamePassword(credentialsId: credentialsId, passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
                sh """
                    curl -v -u "$USERNAME:$PASSWORD" \
                         --upload-file "${filePath}" \
                         "${nexusUrl}"
                """
            }
        }
    }
}

pipeline {
    agent any

    stages {
        stage('Upload Files to Nexus') {
            steps {
                script {
                    uploadAllFilesToNexus(
                        baseDir: 'a', // This is inside the Jenkins workspace
                        repoUrl: 'http://localhost:8081/repository',
                        repoPath: 'iranRepo',
                        credentialsId: 'nexus3'
                    )
                }
            }
        }
    }
}
