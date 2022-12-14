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
            sh "split -l 50 -a 1 -x tests-full.txt test "

        }
    }

    stage('E2E tests'){

        parallel{
                stage('Tests0'){
                    steps{
                        sh "mkdir testa"
                        sh "cp -R test/* testa"
                        sh "cp test0 testa/tests.txt"
                        dir('testa'){
                        sh "java -cp simulator.jar:analytics.jar:telemetry.jar com.lidar.simulation.Simulator"
                        }
                    }
                }
                stage('Tests1'){
                    steps{
                        sh "mkdir testb"
                        sh "cp -R test/* testb"
                        sh "cp test1 testb/tests.txt"
                        dir('testb'){
                        sh "java -cp simulator.jar:analytics.jar:telemetry.jar com.lidar.simulation.Simulator"
                        }
                    }
                }
                stage('Tests2'){
                    steps{
                        sh "mkdir testc"
                        sh "cp -R test/* testc"
                        sh "cp test2 testc/tests.txt"
                        dir('testc'){
                        sh "java -cp simulator.jar:analytics.jar:telemetry.jar com.lidar.simulation.Simulator"
                        }
                    }
                }
                                stage('Tests3'){
                    steps{
                        sh "mkdir testd"
                        sh "cp -R test/* testd"
                        sh "cp test3 testd/tests.txt"
                        dir('testd'){
                        sh "java -cp simulator.jar:analytics.jar:telemetry.jar com.lidar.simulation.Simulator"
                        }
                    }
                }
                stage('Tests4'){
                    steps{
                        sh "mkdir teste"
                        sh "cp -R test/* teste"
                        sh "cp test4 teste/tests.txt"
                        dir('testa'){
                        sh "java -cp simulator.jar:analytics.jar:telemetry.jar com.lidar.simulation.Simulator"
                        }
                    }
                }
                stage('Tests5'){
                    steps{
                        sh "mkdir testf"
                        sh "cp -R test/* testf"
                        sh "cp test5 testf/tests.txt"
                        dir('testf'){
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
            cleanWs()
            }
        }
    }

    }
}
