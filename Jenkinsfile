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


        stage('Fetch SonarQube Metrics') {
            steps {
                script {
                    // SonarQube API'den metrikleri al
                    def metrics = 'bugs,vulnerabilities,code_smells'
                    def sonarMetrics = sh(script: "curl -u ${SONARQUBE_TOKEN}: '${SONARQUBE_HOST}/api/measures/component?component=${SONARQUBE_PROJECT_KEY}&metricKeys=${metrics}'", returnStdout: true).trim()
                    env.SONAR_METRICS = sonarMetrics
                }
            }
        }
    }

    post {
        always {
            script {
                // SonarQube Quality Gate ve metrik sonuçlarını al
                def qg = currentBuild.rawBuild.result
                def metrics = env.SONAR_METRICS ?: 'Metrikler alınamadı'

                // E-posta gönder
                mail to: "${EMAIL_RECIPIENTS}",
                     subject: "SonarQube Raporu - ${qg}",
                     body: "SonarQube Quality Gate sonucu: ${qg}\nSonarQube Metrikleri:\n${metrics}\nPipeline Logları için Jenkins'e bakın."
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
