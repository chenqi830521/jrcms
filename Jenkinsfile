/**
* Requirement:
* 1. Kubernetes cluster
* 2. Credentials: 
*    - docker-hub
*    - aws-eb-key
* 3. AWS Elastic Benstalk in us-east-1 region
*    - Application: jrcms
*    - Environment: jrcms-test, jrcms-staging and jrcms-production
**/

podTemplate(
    containers: [containerTemplate(name: 'docker', image: 'docker', command: 'cat', ttyEnabled: true),
                     containerTemplate(name: 'eb', image: 'mini/eb-cli', command: 'cat', ttyEnabled: true)],
    volumes: [hostPathVolume(hostPath: '/var/run/docker.sock', mountPath: '/var/run/docker.sock')]
  ) {

    node(POD_LABEL) {
      
        def image
        withCredentials([usernamePassword(credentialsId: 'docker-hub', usernameVariable: 'USER', passwordVariable: 'PASSWD')]) {
            image = "${USER}/jrcms:${currentBuild.number}"
        }
        
        stage('Build and Test') {
            checkout scm // 相当于git 'https://github.com/chenqi830521/jrcms.git'
            container('docker') {
                sh "docker build -t ${image} ." //sh 'docker build -t chenqi830521/jrcms:rogerjrcms .'
            }
        }
        
        if (env.BRANCH_NAME == 'master') {
            stage('Push Docker image') {
                withCredentials([usernamePassword(credentialsId: 'docker-hub', usernameVariable: 'USER', passwordVariable: 'PASSWD')]) {
                container('docker') {
                    sh "docker login --username ${USER} --password ${PASSWD}"
                    sh "docker push ${image}"
                }
                }
            }
            
            stage("Deploy to test environment") {
                deployToEB('test')
            }
            
            stage("Integration test to test environment") {  
                smokeTest('test')
            }
            /*
            stage("Deploy to staging environment") {
                deployToEB('staging')
            }
            
            stage("Integration test to staging environment") {  
                smokeTest('staging')
            }
            
            stage("Deploy to production environment") {
                deployToEB('production')
            }*/
        }
    }
}


def smokeTest(environment) {     
    container('eb') {
    	// 添加测试内容
    }
}


def deployToEB(environment) {
    // checkout scm // 这里可以省略
    withCredentials([usernamePassword(credentialsId: 'aws-eb-key', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
            container('eb') {
                withEnv(["AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID}", "AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY}", "AWS_REGION=us-east-1"]) {
                    dir("deployment") {
                    sh "sh generate-dockerrun.sh ${currentBuild.number}"
                    sh "eb deploy JrcmsRoger-env-${environment} -l ${currentBuild.number}"
                    }
                }
            }
    }
}
