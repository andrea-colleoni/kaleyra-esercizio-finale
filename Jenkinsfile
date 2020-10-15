def getBuildDate() {
    return new Date().format('yyyy-MM-dd HH:mm:ss')
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
        stage("Clean WS") {
            steps {
                cleanWs()
            }
        }
        stage("git checkout") {
            steps {
                script {
                    // Checkout the repository and save the resulting metadata
                    def scmVars = checkout(
                        [
                            $class: 'GitSCM', branches: [[name: '*/main']], 
                            doGenerateSubmoduleConfigurations: false, 
                            extensions: [], 
                            submoduleCfg: [], 
                            userRemoteConfigs: 
                            [
                                [
                                    credentialsId: 'github-andrea', 
                                    url: 'https://github.com/andrea-colleoni/kaldevops-projects'
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
            when { changeset "sts/*"}
            steps {
                withMaven(jdk: 'JDK 14', maven: 'Maven 3.6.3') {
                    bat 'mvn -f ./sts/devops-project/pom.xml test'
                }
            }
        }
        stage('maven install+deploy') {
            when { changeset "sts/*"}
            steps {
                withMaven(jdk: 'JDK 14', maven: 'Maven 3.6.3') {
                    bat 'mvn -f ./sts/devops-project/pom.xml install'
                }
            }
        }
        stage('git tag') {
            when { changeset "sts/*"}
            steps {
                bat "git add build-info.md"
                bat "git commit -m \"jenkins build ${currentBuild.number}\" -a"
                bat "git tag v-${currentBuild.number}"
                bat "git push origin v-${currentBuild.number}"
            }
        }
        stage('deploy') {
            when { changeset "sts/*"} 
            steps {
                echo 'deploy'
                ansiblePlaybook colorized: true, 
                    credentialsId: 'andre-private-key', 
                    installation: 'ansible-2.9', 
                    inventory: '/inventory', 
                    limit: 'prod', 
                    playbook: '/plab.yml', 
                    sudo: true
            }
        }
    }
}
