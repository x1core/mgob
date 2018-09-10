library identifier: 'jenkins-pipelines@master', retriever: modernSCM(
  [$class: 'GitSCMSource',
   remote: 'git@github.com:x1core/mgob.git'])

pipeline {

    agent {
        label 'ja_lin01'
    }

    stages {
        stage("setup tools") {
            steps {
                sh 'curl --insecure -LO https://storage.googleapis.com/container-structure-test/latest/container-structure-test-linux-amd64 && chmod +x container-structure-test-linux-amd64'   
            }
        }
        stage("jdk8 build") {
            steps {
                sh 'cd src/docker/jdk8 && curl -L -b "oraclelicense=a" http://download.oracle.com/otn-pub/java/jdk/8u181-b13/96a7b8442fe848ef90c96a2fad6ed6d1/jdk-8u181-linux-x64.tar.gz -O'
                sh 'cd src/docker/jdk8 && curl -LO "http://download.oracle.com/otn-pub/java/jce/8/jce_policy-8.zip" -H "Cookie: oraclelicense=accept-securebackup-cookie"'

                script {
                    docker.withRegistry('https://mtr.external.otc.telekomcloud.com','DOCKER_LOGIN_MTR') {
                        def baseImage = docker.build("mtr.external.otc.telekomcloud.com/magentahaus/mgob", "--build-arg HTTP_PROXY='http://proxy-vip:3128' --build-arg HTTPS_PROXY='http://proxy-vip:3128' .")
                        sh './src/docker/jdk8/test/run_test.sh'

                        if ('master' == env.BRANCH_NAME) {
                            baseImage.push('latest')
                        }
                    }
                }
            }
        }

        
    }
    post {
        success {
            updateGitlabCommitStatus name: currentBuild.displayName, state: 'success'
        }
        failure {
            updateGitlabCommitStatus name: currentBuild.displayName, state: 'failed'
        }
        aborted {
            updateGitlabCommitStatus name: currentBuild.displayName, state: 'canceled'
        }
        always {
            //archiveArtifacts artifacts: "src/docker/aaxapplication/test/reports/**/*.html", fingerprint: true
            archiveArtifacts artifacts: "linux-patch-baseline/reports/**/*.html", fingerprint: true
            //junit "src/docker/aaxapplication/test/reports/**/*.xml"
            junit "linux-patch-baseline/reports/**/*.xml"
            //cleanWs()
        }
    }
}
