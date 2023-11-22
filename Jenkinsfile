pipeline {
    agent any

    environment {
        PATH = "$PATH:$HOME/.dotnet/tools"
        SONARQUBE_HOST = 'http://192.168.176.24:9000'
        SONARQUBE_TOKEN = 'sqp_104bdde603f13ded777f0e840264dc6dcf6177aa'
        EMAIL_RECIPIENTS = 'taha.tangulu@virgosol.com'
    }
    triggers {
        pollSCM('H/5 * * * *')
    }
    
    stages {
        stage('Checkout from Github') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main' || env.BRANCH_NAME == 'dev' || env.BRANCH_NAME ==~ /VDMKC-.*/) {
                git credentialsId: 'TahaTangulu', url: 'https://github.com/TeamsecABS/vdmk-backend.git', branch: env.BRANCH_NAME
                    }
                }
            }
        } 
        
        stage('Restore') {
            steps {
                sh 'dotnet restore'
            }
        }

        stage('Build & Test') {
            steps {
                sh 'dotnet build'
                sh 'dotnet test'
            }
        }
        
        stage('Install .NET Core SonarScanner and Run Analysis') {

            steps {
                script {
                    sh 'dotnet tool install --global dotnet-sonarscanner || true'
                    sh 'echo "##vso[task.setvariable variable=PATH;]$PATH:$HOME/.dotnet/tools"'
                    sh "dotnet sonarscanner begin /k:'vdmk-backend' /d:sonar.host.url='${SONARQUBE_HOST}' /d:sonar.login='${SONARQUBE_TOKEN}'"
                    sh 'dotnet build'
                    sh "dotnet sonarscanner end /d:sonar.login='${SONARQUBE_TOKEN}'"
                }
            }
        }

        stage('SonarQube Quality Gate') {
            steps {
                script {
                    def qg = waitForQualityGate() // Quality Gate sonucunu bekler
                    if (qg.status != 'OK') {
                        currentBuild.result = 'FAILURE'
                        error "Quality Gate failed: ${qg.status}"
                    }
                }
            }
        }
    }

    post {
        always {
            script {
                // SonarQube Quality Gate durumunu almak
                def qg = currentBuild.rawBuild.result
                // E-posta gönder
                mail to: "${EMAIL_RECIPIENTS}",
                     subject: "SonarQube Quality Gate Durumu: ${qg}",
                     body: "SonarQube Quality Gate sonucu: ${qg}\nPipeline Logları için Jenkins'e bakın."
            }
            echo 'Pipeline execution complete!'
        }
        failure {
            mail to: "${EMAIL_RECIPIENTS}",
                 subject: "Pipeline Hatası",
                 body: "Pipeline sırasında bir hata oluştu. Lütfen Jenkins loglarına bakınız."
        }
    }
}
