def uploadSpecificFilesToNexus(String baseDir, String repoUrl, String repoPath, String credentialsId, String fileName) {
    def currentDate = new Date().format('dd-MM-yyyy')

    dir(baseDir) {
        def files = sh(returnStdout: true, script: "find . -type f -name '${fileName}'").trim().split('\n')

        files.each { filePath ->
            def trimmedPath = filePath.tokenize('/').last()
            def nexusUrl = "${repoUrl}/${repoPath}/${currentDate}/feature_test/${trimmedPath}"

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
        stage('Upload Specific Files') {
            steps {
                script {
                    uploadSpecificFilesToNexus(
                        baseDir: 'src', // replace with your base folder
                        repoUrl: 'https://your-nexus-url/repository', // replace with your Nexus URL
                        repoPath: 'cons', // example repo path
                        credentialsId: 'nexus-creds-id', // Jenkins credentials ID
                        fileName: 'myfile.json' // file to look for
                    )
                }
            }
        }
    }
}
