pipeline {
    agent any

    parameters {
       // string(defaultValue: 'v2', description: '', name: 'buildVersion')
        choice(
            choices: 'Rollingupdate\nBlue-Green',
            description: 'Deployment Type',
            name: 'REQUESTED_ACTION')
    }
    environment {
    // Environment variable identifiers need to be both valid bash variable
    // identifiers and valid Groovy variable identifiers. If you use an invalid
    // identifier, you'll get an error at validation time.
    // Right now, you can't do more complicated Groovy expressions or nesting of
    // other env vars in environment variable values, but that will be possible
    // when https://issues.jenkins-ci.org/browse/JENKINS-41748 is merged and
    // released.
        service = "my-service"
        deployment = "nodejs"
        DEPLOYMENTFILE = "deploy-green.yaml"
        VERSION= "${BUILD_ID}"
        image= "pavanraj29/nodejs-app-demo"
     }
    
    stages {
        stage("build") {
            steps {
                sh 'rm -rf HM-Demo'
                sh 'git clone https://github.com/pavaraj29/HM-Demo.git'
            }
        }
        stage("Docker image build") {
            steps {
                sh 'cd HM-Demo &&  sudo docker build -t nodejs-image-new .'
            }
        }
        stage("Docker image tag") {
            steps {
                sh 'sudo  docker tag nodejs-image-new ${image}:${VERSION}'
                //sh 'sudo  docker tag nodejs-image-new ${image}'
            }
        }
        stage("Docker image push") {
            steps {
                withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId:'Dockercreds', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']]) {
                sh '''sudo docker login -u $USERNAME -p $PASSWORD
                sudo docker push ${image}:${VERSION}
                cd HM-Demo
                sed -i -e 's/nodejs-app-demo/nodejs-app-demo:'${VERSION}'/g' patch.yaml
                sed -i -e 's/appVersion/"'${VERSION}'"/g' patch-svc.yaml
                '''
                }
            }
        }
        
        stage("Rollingupdate Deployment") {
             when {
                // Only say hello if a "greeting" is requested
                expression { params.REQUESTED_ACTION == 'Rollingupdate' }
            }
            steps {
                sh '''
                cd HM-Demo
                kubectl patch deployment ${deployment} --patch "$(cat patch.yaml)"
                '''
            }
        }
        stage("Blue-green Deployment") {
            when {
                // Only say hello if a "greeting" is requested
                expression { params.REQUESTED_ACTION == 'Blue-Green' }
            }
            steps {
                sh '''
                cd HM-Demo
                kubectl apply -f ${DEPLOYMENTFILE}
                kubectl patch svc ${service} -p "$(cat patch-svc.yaml)"
                '''
            }
        }
     }  
}
