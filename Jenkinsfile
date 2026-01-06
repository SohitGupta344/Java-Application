pipeline {
    agent any

    environment {
        MAVEN_OPTS   = "-Xmx1024m"
        JAVA_HOME    = "C:\\Program Files\\Eclipse Adoptium\\jdk-17.0.17.10-hotspot"
        TOMCAT_HOME  = "C:\\Tools\\tomcat\\apache-tomcat-9.0.113"
        WAR_NAME     = "JavaLoginShowcase-1.0.0.war"
        PATH         = "${env.JAVA_HOME}\\bin;${env.PATH}"
        SONAR_HOST   = "http://localhost:9000"
        SONAR_TOKEN  = "sqp_1ec77f76707017438dea02caf728439b66f849bd"
    }

    stages {

        // ==================================
        // STAGE 1: SONARQUBE ANALYSIS
        // ==================================
        stage('Static Code Analysis - SonarQube') {
            steps {
                echo "Running SonarQube Analysis"

                powershell """
                sonar-scanner `
                  -Dsonar.projectKey=java-application `
                  -Dsonar.projectName=java-application `
                  -Dsonar.sources=src `
                  -Dsonar.exclusions=node_modules/**,build/** `
                  -Dsonar.host.url=${SONAR_HOST} `
                  -Dsonar.token=${SONAR_TOKEN}
                """
            }
        }

        // ==================================
        // STAGE 2: MAVEN BUILD (WAR)
        // ==================================
        stage('Build WAR using Maven') {
            steps {
                echo "Java & Maven Version Check"

                powershell """
                echo JAVA_HOME=%JAVA_HOME%
                java -version
                mvn -version
                """

                echo "Maven Clean Package"
                powershell "mvn clean package"
            }

            post {
                success {
                    archiveArtifacts artifacts: 'target/*.war', fingerprint: true
                }
            }
        }

        // ==================================
        // STAGE 3: DEPLOY TO TOMCAT
        // ==================================
        stage('Deploy to Tomcat') {
            steps {
                echo "Stopping Tomcat"
                powershell """
                & "${TOMCAT_HOME}\\bin\\shutdown.bat"
                Start-Sleep -Seconds 10
                """

                echo "Deploying WAR"
                powershell """
                Copy-Item "target\\${WAR_NAME}" `
                          "${TOMCAT_HOME}\\webapps\\JavaLoginShowcase.war" -Force
                """

                echo "Starting Tomcat"
                powershell """
                & "${TOMCAT_HOME}\\bin\\startup.bat"
                """
            }
        }
    }

    post {
        success {
            echo "✅ Application deployed successfully to Tomcat"
        }
        failure {
            echo "❌ Pipeline failed"
        }
    }
}
