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
        cleanWs()
        sh "mvn verify"
      }
    }

    stage('Files for E2E tests'){
        steps{
            sh "mkdir -p test"  
            
            withCredentials([usernamePassword(credentialsId: 'aleks_jfrog', passwordVariable: 'password', usernameVariable: 'myUser')]) {
            script{
            //get newest versions
            TELEMETRY_VERSION = sh(returnStdout: true, script: "curl -u $myUser:$password http://artifactory:8082/artifactory/exam-libs-release-local/com/lidar/telemetry/maven-metadata.xml | grep '<version>' | tail -1 | grep -o '[0-9].[0-9].[0-9]'").trim()
            ANALYTICS_VERSION = sh(returnStdout: true, script: "curl -u $myUser:$password http://artifactory:8082/artifactory/exam-libs-release-local/com/lidar/analytics/maven-metadata.xml | grep '<version>' | tail -1 | grep -o '[0-9].[0-9].[0-9]'").trim()
            }              
            sh "curl -u $myUser:$password http://artifactory:8082/artifactory/exam-libs-release-local/com/lidar/telemetry/${TELEMETRY_VERSION}/telemetry-${TELEMETRY_VERSION}.jar --output test/telemetry.jar"
            sh "curl -u $myUser:$password http://artifactory:8082/artifactory/exam-libs-release-local/com/lidar/analytics/${ANALYTICS_VERSION}/analytics-${ANALYTICS_VERSION}.jar --output test/analytics.jar"
        
            }
            sh "cp target/simulator-99-SNAPSHOT.jar test/simulator.jar"
            sh "split -n 4 -a 1 -x tests-full.txt test"

        }
    }

    stage('E2E tests'){

        parallel{
                stage('Tests0'){
                    steps{
                        sh "cp -R test/* test0"
                        sh "cp test0 test0/tests.txt"
                        dir('test'){
                        sh "java -cp simulator.jar:analytics.jar:telemetry.jar com.lidar.simulation.Simulator"
                        }
                    }
                }
                stage('Tests1'){
                    steps{
                        sh "cp -R test/* test1"
                        sh "cp test1 test1/tests.txt"
                        dir('test'){
                        sh "java -cp simulator.jar:analytics.jar:telemetry.jar com.lidar.simulation.Simulator"
                        }
                    }
                }
                stage('Tests2'){
                    steps{
                        sh "cp -R test/* test2"
                        sh "cp test2 test2/tests.txt"
                        dir('test'){
                        sh "java -cp simulator.jar:analytics.jar:telemetry.jar com.lidar.simulation.Simulator"
                        }
                    }
                }
                                stage('Tests3'){
                    steps{
                        sh "cp -R test/* test3"
                        sh "cp test3 tes3/tests.txt"
                        dir('test'){
                        sh "java -cp simulator.jar:analytics.jar:telemetry.jar com.lidar.simulation.Simulator"
                        }
                    }
                }
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
