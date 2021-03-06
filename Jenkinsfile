def getBuildDate() {
    return new Date().format('yyyy-MM-dd HH:mm:ss')
}
def getBuildEnv() {
    return "collaudo"
}

pipeline {
    agent any

    stages {
        stage('Project info') {
            steps {
                echo "Project: ${currentBuild.projectName}"
                echo "Build #: ${currentBuild.number}"
            }
        }        
        stage("git checkout") {
            steps {
                script {
                    // Checkout the repository and save the resulting metadata
                    def scmVars = checkout(
                        [
                            $class: 'GitSCM', branches: [[name: "*/$buildEnv"]], 
                            doGenerateSubmoduleConfigurations: false, 
                            extensions: [], 
                            submoduleCfg: [], 
                            userRemoteConfigs: 
                            [
                                [
                                    url: 'file:///C:/Users/andre/Desktop/Temp/Corsi/Kaleyra-DevOps/esercizio-finale'
                                ]
                            ]
                        ]
                    )
                    env.GIT_COMMIT = scmVars.GIT_COMMIT
                } 
                bat 'dir'
            }
        }
        stage("write build info") {
            steps {
                writeFile file: 'build-info.md', text: """# Informazioni build

                    - Progetto: ${currentBuild.projectName}-${currentBuild.number}
                    - Data build: $buildDate
                    - SHA1 git commit: ${env.GIT_COMMIT}"""
            }
        }
        stage('maven compile') {
            when { changeset "EsercizioFinale/*"}
            steps {
                withMaven(jdk: 'JDK 14', maven: 'Maven 3.6.3') {
                    bat 'mvn -f ./sts/devops-project/pom.xml test'
                }
            }
        }                
        stage("Clean WS") {
            steps {
                cleanWs()
            }
        }        
    }
}
