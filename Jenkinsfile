//@Library('Utilities') _
import groovy.json.JsonSlurper
import hudson.model.*
def BuildVersion
def Current_version
def NextVersion
def dev_rep_docker = 'devopsint/dev'
def colons = ':'
def module = 'intapi'
def underscore = '_'

pipeline {
    options {
        timeout(time: 30, unit: 'MINUTES')
    }
    agent { label 'slave' }
    stages {
         stage('Checkout') {
             steps {
                 script {
                     node('master'){
                         dir('Release') {
                             deleteDir()
                             checkout([$class: 'GitSCM', branches: [[name: 'gadi']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'INT_API', url: "https://github.com/gadigamburg/Release.git"]]])
                             println("Test")
                             path_json_file = sh(script: "pwd", returnStdout: true).trim() + '/' + 'release' + '.json'
                             println("path_json_file: $path_json_file")
                             Current_version = Return_Json_From_File("$path_json_file").release.services.intapi.version
                             println("Current_version: $Current_version")
                         }
                     }
                     dir('INT_API') {
                         deleteDir()
                         checkout([$class: 'GitSCM', branches: [[name: 'gadi']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'INT_API', url: "https://github.com/gadigamburg/INT_API.git"]]])
                         Commit_Id = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                         BuildVersion = Current_version + '_' + Commit_Id
                         println("BuildVersion: $BuildVersion")
                         last_digit_current_version = sh(script: "echo $Current_version | cut -d'.' -f3", returnStdout: true).trim()
                         NextVersion = sh(script: "echo $Current_version | cut -d. -f1", returnStdout: true).trim() + '.' + sh(script: "echo $Current_version |cut -d'.' -f2", returnStdout: true).trim() + '.' + (Integer.parseInt(last_digit_current_version) + 1)
                         println("Checking the build version: $BuildVersion")
                     }
                 }
             }
         }
         stage('UT') {
             steps {
                 println('UT will be added soon GADI')
             }
         }
         stage('Build') {
             steps {
                 script {
                     dir('INT_API') {
                         try {

                           docker.build("$module$colons$BuildVersion")
                           println("The build image is successfully")  

                         }
                         catch (exception) {
                             println "The image build is failed"
                             currentBuild.result = 'FAILURE'
                             throw exception
                         }
                     }
                 }
             }
         }
         stage('Push image to repository'){
             steps{
                 script{
                     try{
                         withCredentials([usernamePassword(credentialsId: 'docker_hub', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                            println("DockerUser: ${DOCKER_USERNAME}")
                            println("DockerPass: ${DOCKER_PASSWORD}")
                            sh "docker login -u=${DOCKER_USERNAME} -p=${DOCKER_PASSWORD}"
                            sh "docker tag $module$colons$BuildVersion $dev_rep_docker$colons$module$underscore$BuildVersion"
                            sh "docker push $dev_rep_docker$colons$module$underscore$BuildVersion"
                         }
                     }
                     catch (exception){
                         println "Pushing image to DockerHub failed"
                         currentBuild.result = 'FAILURE'
                         throw exception
                     }
                 }
             }
         }
         stage('Push tag version to repository'){
             steps{
                 script{
                    println('Tag version will be added soon')
                 }
             }
         }
    }
}
def Return_Json_From_File(file_name){
    return new JsonSlurper().parse(new File(file_name))
}