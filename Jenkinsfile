@Library('jenkins-helpers@v0.1.16') _

def label = "spark-metrics-${UUID.randomUUID().toString().substring(0, 5)}"

podTemplate(label: label,
            containers: [containerTemplate(name: 'maven',
                                           image: 'maven:3.5.2-jdk-8',
                                           resourceRequestCpu: '100m',
                                           resourceLimitCpu: '2000m',
                                           resourceRequestMemory: '3000Mi',
                                           resourceLimitMemory: '3000Mi',
                                           ttyEnabled: true,
                                           command: '/bin/cat -')],
            volumes: [secretVolume(secretName: 'maven-credentials', mountPath: '/maven-credentials')]) {

    node(label) {
        def imageTag
        container('jnlp') {
            stage('Checkout') {
                checkout(scm)
                imageTag = sh(returnStdout: true,
                              script: 'echo \$(date +%Y-%m-%d)-\$(git rev-parse --short HEAD)').trim()
            }
        }
        container('maven') {
            stage('Install Maven credentials') {
                sh('cp /maven-credentials/settings.xml /root/.m2')
                sh('mkdir -p /home/jenkins/.m2 && cp /maven-credentials/settings.xml /home/jenkins/.m2/')
            }
            stage('Test') {
                sh('mvn test || true')
                junit(allowEmptyResults: false, testResults: '**/target/surefire-reports/*.xml')
                summarizeTestResults()
                sh('mvn verify -DskipTests')
            }
            //if (env.BRANCH_NAME == 'master') {
                stage('Deploy') {
                    sh('mvn deploy')
                }
            //}
        }
    }
}
