pipeline {
  agent any

  tools {
    maven 'maven'
  }

  stages {
    stage('Build') {
      steps {
        withMaven(maven : 'maven') {
          sh "mvn package"
        }
      }
    }

    stage ('OWASP Dependency-Check Vulnerabilities') {
      steps {
        withMaven(maven : 'maven') {
          sh 'mvn dependency-check:check'
        }

        dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
      }
    }

    stage ('PMD SpotBugs') {
      steps {
        withMaven(maven : 'maven') {
          sh 'mvn pmd:pmd pmd:cpd spotbugs:spotbugs'
        }

        recordIssues enabledForFailure: true, tool: spotBugs()
        recordIssues enabledForFailure: true, tool: cpd(pattern: '**/target/cpd.xml')
        recordIssues enabledForFailure: true, tool: pmdParser(pattern: '**/target/pmd.xml')
      }
    }

    stage ('ZAP') {
      steps {
        withMaven(maven : 'maven') {
          sh 'mvn zap:analyze'
          publishHTML (target: [
                allowMissing: false,
                alwaysLinkToLastBuild: false,
                keepAll: true,
                reportDir: 'target/zap-reports',
                reportFiles: 'zapReport.html',
                reportName: "ZAP report"
              ])
        }
      }
    }

    stage('SonarQube analysis') {
      steps {
        withSonarQubeEnv(credentialsId: 'sonarqube-secret', installationName: 'sonarqube-server') {
          withMaven(maven : 'maven') {
            sh 'mvn sonar:sonar -Dsonar.dependencyCheck.jsonReportPath=target/dependency-check-report.json -Dsonar.dependencyCheck.xmlReportPath=target/dependency-check-report.xml -Dsonar.dependencyCheck.htmlReportPath=target/dependency-check-report.html -Dsonar.java.pmd.reportPaths=target/pmd.xml -Dsonar.java.spotbugs.reportPaths=target/spotbugsXml.xml -Dsonar.zaproxy.reportPath=target/zap-reports/zapReport.xml -Dsonar.zaproxy.htmlReportPath=target/zap-reports/zapReport.html'
          }
        }
      }
    }
  }
}
