#!groovy
@Library('github.com/cloudogu/ces-build-lib@caa9632')
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

            String prArgs = ""
            if (isPullRequest()) {
                prArgs = "-Dsonar.pullrequest.base=master" +
                         "-Dsonar.pullrequest.branch=${env.BRANCH_NAME}" +
                         "-Dsonar.pullrequest.key=${env.CHANGE_ID}" +
                         "-Dsonar.pullrequest.provider=GitHub" +
                         "-Dsonar.pullrequest.github.repository=new Git(this).gitHubRepositoryName"
            }

            withSonarQubeEnv('sonarcloud.io') {
                mvn "${env.SONAR_MAVEN_GOAL} " +
                        "-Dsonar.host.url=${env.SONAR_HOST_URL} " +
                        "-Dsonar.login=${env.SONAR_AUTH_TOKEN} " +
                        "${prArgs}"
            }
        }
    }

    // Archive Unit and integration test results, if any
    junit allowEmptyResults: true, testResults: '**/target/failsafe-reports/TEST-*.xml,**/target/surefire-reports/TEST-*.xml'
}

// Init global vars in order to avoid NPE
String cesFqdn = ''
String cesUrl = ''
