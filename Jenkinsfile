pipeline {
    options {
        disableConcurrentBuilds()
        gitLabConnection gitLabConnection: 'gitlab', jobCredentialId: 'gitlab_apitoken'
        timeout(activity: true, time: 5)
    }
    
    tools {
        jdk 'jdk8'
        maven '3.6.2'
    }
    agent any

    stages {
 
    stage('Build') {
      steps {
        sh "mvn verify"
      }
    }

    stage('E2E tests'){
        steps{
            sh "mkdir -p test"  
            
            withCredentials([usernamePassword(credentialsId: 'aleks_jfrog', passwordVariable: 'password', usernameVariable: 'myUser')]) {
            script{
            //get newest versions
            TELEMETRY_VERSION = sh(returnStdout: true, script: "curl -u $myUser:$password http://artifactory:8082/artifactory/exam-libs-release-local/com/lidar/telemetry/maven-metadata.xml | grep '<version>' | tail -1 | grep -o '[0-9].[0-9].[0-9]'").trim()
            ANALYTICS_VERSION = sh(returnStdout: true, script: "curl -u $myUser:$password http://artifactory:8082/artifactory/exam-libs-release-local/com/lidar/analytics/maven-metadata.xml | grep '<version>' | tail -1 | grep -o '[0-9].[0-9].[0-9]'").trim()
            }              
            sh "curl -u $myUser:$password http://artifactory:8082/artifactory/exam-libs-release-local/com/lidar/telemetry/${TELEMETRY_VERSION}/telemetry-${TELEMETRY_VERSION}.jar --output test/telemetry.jar"
            sh "curl -u $myUser:$password http://artifactory:8082/artifactory/exam-libs-release-local/com/lidar/analytics/${ANALYTICS_VERSION}/telemetry-${ANALYTICS_VERSION}.jar --output test/analytics.jar"
        
            }
            sh "cp target/simulator-99-SNAPSHOT.jar test/simulator.jar"
            sh "cp tests-full.txt test/tests.txt"
            dir('test'){
                sh "java -cp simulator.jar:analytics.jar:telemetry.jar com.lidar.simulation.Simulator"
            }
            sh "rm -r test"

        }
    }

    stage('Publish') {
        steps {
            configFileProvider([configFile(fileId: 'exam_maven_settings', variable: 'SETTINGS')]) {
            sh "mvn deploy -s $SETTINGS -Dmaven.test.skip"
            }
        }
    }

    }
}
