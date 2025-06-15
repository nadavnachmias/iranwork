pipeline {
    agent any

    stages {
        stage('Delete Folders Older Than 7 Days') {
            steps {
                script {
                    def repoUrl     = 'http://localhost:8081'
                    def repoName    = 'iranRepo'
                    def credentials = 'nexus3'
                    def threshold   = new Date() - 7

                    withCredentials([usernamePassword(credentialsId: credentials, usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                        def json = sh(
                            returnStdout: true,
                            script: """curl -k -s -u "$USERNAME:$PASSWORD" "${repoUrl}/service/rest/v1/assets?repository=${repoName}" """
                        ).trim()

                        def data = readJSON text: json

                        def folders = data.items
                            .collect { it.path.tokenize('/')[0] }
                            .findAll { it ==~ /\\d{2}-\\d{2}-\\d{4}/ }
                            .unique()

                        folders.each { folder ->
                            try {
                                def date = Date.parse('dd-MM-yyyy', folder)
                                if (date.before(threshold)) {
                                    def deleteUrl = "${repoUrl}/repository/${repoName}/${folder}/"
                                    echo "🗑️ Deleting: ${deleteUrl}"
                                    sh """curl -k -u "$USERNAME:$PASSWORD" -X DELETE "${deleteUrl}" """
                                } else {
                                    echo "✅ Keeping: ${folder}"
                                }
                            } catch (ignored) {
                                echo "⚠️ Skipping invalid: ${folder}"
                            }
                        }
                    }
                }
            }
        }
    }
}
