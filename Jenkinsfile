pipeline {
    agent any

    environment {
        REPO_URL = "http://localhost:8081/repository/iranRepo"
        CREDENTIALS_ID = "nexus3"
    }

    stages {
        stage('Delete Folders Older Than 7 Days') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: "${CREDENTIALS_ID}", passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
                        // List top-level folders
                        def rawList = sh(script: """
                            curl -sk -u "$USERNAME:$PASSWORD" "$REPO_URL/" | grep -oP '(?<=href=")[0-9]{2}-[0-9]{2}-[0-9]{4}/(?=")' | sed 's#/##'
                        """, returnStdout: true).trim()

                        if (!rawList) {
                            echo "⚠️ No date-based folders found."
                            return
                        }

                        def folderList = rawList.split("\\n")
                        def threshold = new Date() - 7

                        folderList.each { folder ->
                            try {
                                def folderDate = Date.parse("dd-MM-yyyy", folder)
                                if (folderDate.before(threshold)) {
                                    echo "🗑 Deleting folder: $folder"
                                    sh """
                                        curl -sk -u "$USERNAME:$PASSWORD" -X DELETE "$REPO_URL/$folder/"
                                    """
                                } else {
                                    echo "✅ Keeping folder: $folder (not old enough)"
                                }
                            } catch (Exception e) {
                                echo "⛔ Skipping invalid folder name: $folder"
                            }
                        }
                    }
                }
            }
        }
    }
}
