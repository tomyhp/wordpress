pipeline {
    agent any
    stages {
        stage ('Build wordpress') {
            steps {
                sh '''
                    sudo docker build -t tomyhp/wordpress:$GIT_BRANCH-$BUILD_ID -f worpress/Dockerfile .
                    sudo docker login -u tomyhp -p$DOCKER_TOKEN
                    sudo docker push tomyhp/wordpress:$GIT_BRANCH-$BUILD_ID
                '''
                }
            }
        
       
        stage ('Change manifest file and send') {
            steps {
                script {
		        if (BRANCH_NAME == 'main'){
                sh '''
                    sed -i -e "s/branch/$GIT_BRANCH/" Kube-production/wordpress/wordpress-deployment.yml
                    sed -i -e "s/appversion/$BUILD_ID/" Kube-production/wordpress/wordpress-deployment.yml
                    tar -czvf manifest-production.tar.gz Kube-production/*
                '''
                sshPublisher(
                    continueOnError: false, 
                    failOnError: true,
                    publishers: [
                        sshPublisherDesc(
                            configName: "kube-master-tomy",
                            transfers: [sshTransfer(sourceFiles: 'manifest-production.tar.gz', remoteDirectory: 'jenkins/')],
                            verbose: true
                           )
                        ]
                    )
                } 
                else {
                sh '''
                    sed -i -e "s/branch/$GIT_BRANCH/" Kube-staging/wordpress/wordpress-deployment.yml
                    sed -i -e "s/appversion/$BUILD_ID/" Kube-staging/wordpress/wordpress-deployment.yml
                    tar -czvf manifest-staging.tar.gz Kube-staging/*
                '''
                sshPublisher(
                    continueOnError: false, 
                    failOnError: true,
                    publishers: [
                        sshPublisherDesc(
                            configName: "kube-master-tomy",
                            transfers: [sshTransfer(sourceFiles: 'manifest-staging.tar.gz', remoteDirectory: 'jenkins/')],
                            verbose: true
                            )
                        ]
                     )
                  }
                } 
	        }
        }
          
    
        stage ('Publish Pre-release') {
            when {branch 'staging'}
            steps {	
                sshagent(credentials : ['kube-master-tomy']){
                    sh 'ssh -o StrictHostKeyChecking=no ubuntu@api.lopunya.id tar -xvzf jenkins/manifest-staging.tar.gz'
                    sh 'ssh -o StrictHostKeyChecking=no ubuntu@api.lopunya.id kubectl apply -f /home/ubuntu/Kube-staging/namespace'
                    sh 'ssh -o StrictHostKeyChecking=no ubuntu@api.lopunya.id kubectl apply -f /home/ubuntu/Kube-staging/wordpress'
                    sh 'ssh -o StrictHostKeyChecking=no ubuntu@api.lopunya.id kubectl apply -f /home/ubuntu/Kube-staging/ingress-nginx'
                }
            }
        }    
        stage ('Publish Release') { 
            when {not{branch 'staging'}}
            steps {
                sshagent(credentials : ['kube-master-tomy']){
                    sh 'ssh -o StrictHostKeyChecking=no ubuntu@api.lopunya.id tar -xvzf jenkins/manifest-production.tar.gz'
                    sh 'ssh -o StrictHostKeyChecking=no ubuntu@api.lopunya.id kubectl apply -f /home/ubuntu/Kube-production/namespace'
                    sh 'ssh -o StrictHostKeyChecking=no ubuntu@api.lopunya.id kubectl apply -f /home/ubuntu/Kube-production/wordpress'
                    sh 'ssh -o StrictHostKeyChecking=no ubuntu@api.lopunya.id kubectl apply -f /home/ubuntu/Kube-production/ingress-nginx'
                }
            }
        }
    }  
    
}        
   
            
