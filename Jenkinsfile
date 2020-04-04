mport hudson.model.*
import hudson.EnvVars
import groovy.json.JsonSlurperClassic
import groovy.json.JsonBuilder
import groovy.json.JsonOutput
import java.net.URL

node{

properties([
    buildDiscarder(logRotator(artifactDaysToKeepStr: '7', artifactNumToKeepStr: '10', daysToKeepStr: '7', numToKeepStr: '10')),
    parameters([
        booleanParam(defaultValue: true, description: 'Application Compile ', name: 'Compile'),
        booleanParam(defaultValue: true, description: 'Application Review  ', name: 'Review'),
        booleanParam(defaultValue: true, description: 'Application Test    ', name: 'Test'),
        booleanParam(defaultValue: true, description: 'Application Metrics ', name: 'Metrics'),
        booleanParam(defaultValue: true, description: 'Application Package ', name: 'Package'),
        booleanParam(defaultValue: true, description: 'Docker Image Build  ', name: 'DockerImage')
        ])
    ])

    stage('Checkout'){
        cleanWs()
        git 'https://github.com/rrahul4/DevOpsClassCodes'
    }        
    
    if (params.Compile) {
        stage('Compile'){
            withMaven(maven:'MyMaven'){
            sh 'mvn compile'
            }
        }        
    }

    if (params.Review) {
        stage('Review'){
            withMaven(maven:'MyMaven'){
            sh 'mvn pmd:pmd'
            pmd canComputeNew: false, defaultEncoding: '', healthy: '', pattern: 'target/pmd.xml', unHealthy: ''
            }
        }        
    }

    if (params.Test) {
        stage('Test')  {
            withMaven(maven:'MyMaven'){
            sh 'mvn test'
            junit 'target/surefire-reports/*.xml'
            }
        }        
    }

    if (params.Metrics) {
        stage('Metrics'){
            withMaven(maven:'MyMaven'){
            sh 'mvn cobertura:cobertura -Dcobertura.report.format=xml'
            cobertura autoUpdateHealth: false, autoUpdateStability: false, coberturaReportFile: 'target/site/cobertura/coverage.xml', conditionalCoverageTargets: '70, 0, 0', failUnhealthy: false, failUnstable: false, lineCoverageTargets: '80, 0, 0', maxNumberOfBuilds: 0, methodCoverageTargets: '80, 0, 0', onlyStable: false, sourceEncoding: 'ASCII', zoomCoverageChart: false
            }
        }        
    }

    if (params.Package) {
        stage('Package'){
           withMaven(maven:'MyMaven'){
            sh 'mvn package'
            }
        }        
    }
    
    if (params.DockerImage) {
        stage('Docker Image Build') {
            writeFile file: 'Dockerfile', 
            text: '''FROM tomcat
            Maintainer Rahulkumar Rakhonde
            ADD addressbook.war  /usr/local/tomcat/webapps/
            CMD "catalina.sh" "run"'''
            
            sh label: '',script: '''
            cp $WORKSPACE/target/*.war .
        
            sudo docker build . -t rrahul4/addressbook:$BUILD_NUMBER
            sudo docker push rrahul4/addressbook:$BUILD_NUMBER '''
        }
    }
}
