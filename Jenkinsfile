node{
    
    def mavenHome
    def mavenCMD
    def docker
    def dockerCMD
    def tagName
    
    stage('prepare enviroment'){
        echo 'initialize all the variables'
        mavenHome = tool name: 'myMaven' , type: 'maven'
        mavenCMD = "${mavenHome}/bin/mvn"
        docker = tool name: 'myDocker' , type: 'org.jenkinsci.plugins.docker.commons.tools.DockerTool'
        dockerCMD = "${docker}/bin/docker"
        tagName="3.0"
    }
    
    stage('git code clone'){
        echo 'cloning the code from git repository'
        git 'https://github.com/Sivaops7/insure-me.git'
        
    }
    
    stage('Build the Application'){
        echo "Cleaning... Compiling...Testing... Packaging..."
        //sh 'mvn clean package'
        sh "${mavenCMD} clean package"        
    }
    
    stage('publish test reports'){
        publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '/var/lib/jenkins/workspace/Capstone-Project-Live-Demo/target/surefire-reports', reportFiles: 'index.html', reportName: 'HTML Report', reportTitles: '', useWrapperFileDirectly: true])
    }
    
    stage('Containerize the application'){
        try{
        echo 'Creating Docker image'
        sh "${dockerCMD} build -t sivaops7/insure-me:${tagName} ."
        }
        catch(Exception e){
            echo 'Exception occur in stage Dcker Build'
            currentBuild.result = "FAILURE"
            emailext body: '''Hello,
            We are having issue in Docker Build command .can you please look into the same
            Thanks 
            Jenkins Admin''', subject: 'Attention:${JOB_NAME} is failed.please into the ${BUILD_NAME}', to: 'sivabalana91@gmail.com'
        }
    }
    
    stage('Pushing it ot the DockerHub'){
        echo 'Pushing the docker image to DockerHub'
        withCredentials([string(credentialsId: 'dockerpassword', variable: 'dockerpassword')]) { 
                sh "${dockerCMD} login -u sivaops7 -p ${dockerPassword}"
                sh "${dockerCMD} push sivaops7/insure-me:${tagName}"
        }   
      
    }
        
    stage('Configure and Deploy to the test-server'){
        ansiblePlaybook become: true, credentialsId: 'ansiblekey', disableHostKeyChecking: true, installation: 'myAnsible', inventory: '/etc/ansible/hosts', playbook: 'ansible-playbook.yml'
    }
        
}
