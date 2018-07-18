#!groovy
@Library('github.com/cloudogu/ces-build-lib@3bf5579')
import com.cloudogu.ces.cesbuildlib.*

properties([
        // Don't run concurrent builds, because the ITs use the same port causing random failures on concurrent builds.
        disableConcurrentBuilds()
])

node {

    cesFqdn = findHostName()
    cesUrl = "https://${cesFqdn}"

    Maven mvn = new MavenWrapper(this)

    catchError {

        stage('Checkout') {
            checkout scm
        }

        stage('Build') {
            mvn "-DskipTests clean package"

            // archive artifact
            archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
        }

        parallel(
                test: {
                    stage('Test') {
                        String jacoco = "org.jacoco:jacoco-maven-plugin:0.7.7.201606060606"
                        mvn "${jacoco}:prepare-agent test ${jacoco}:report"
                    }
                },
                integrationTest: {
                    stage('Integration Test') {
                        String jacoco = "org.jacoco:jacoco-maven-plugin:0.7.7.201606060606";
                        mvn "${jacoco}:prepare-agent-integration failsafe:integration-test ${jacoco}:report-integration"
                    }
                }
        )

        stage('SonarQube Analysis') {
          def sonarQube = new SonarCloud(this, 'sonarQubeServerSetupInJenkins', 'schnatterer-github')

          sonarQube.analyzeWith(mvn)

          if (!sonarQube.waitForQualityGateWebhookToBeCalled()) {
            currentBuild.result ='UNSTABLE'
          }
        }
    }

    // Archive Unit and integration test results, if any
    junit allowEmptyResults: true, testResults: '**/target/failsafe-reports/TEST-*.xml,**/target/surefire-reports/TEST-*.xml'
}

// Init global vars in order to avoid NPE
String cesFqdn = ''
String cesUrl = ''
