pipeline {
    agent any
    environment {
        DOME9TOKEN  = credentials('D9API') //'D9API' is stored as secured text, the value is your-api-key-id:your-api-key-secret
        AWS_DEFAULT_REGION =  'us-east-1' //required for the region where the functions are to be deployed
        AWS_ACCESS_KEY_ID = credentials('AWS_KEY')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET')
        GIT_EMAIL = credentials('MY_GIT_EMAIL')
    }
    stages {
        stage('Assess'){ //this "Assess" stage is where we do the static analysis. you can choose your own
            steps {
//                sh 'cloudguard --version' this is a good step when you develope the pipeline. this will make sure that the 
//                      cloudguard cli is already installed. Alternatively you can install it on the agent, as long as the 
//                      agent has docker and npm by using the following command:
//                      npm install -g https://artifactory.app.protego.io/cloudguard-serverless-plugin.tgz
                sh 'cd $WORKSPACE' //generally superfluous but just in case
                sh 'git config --global user.email $GIT_EMAIL' 
                withCredentials([usernamePassword(credentialsId: 'newgithub', passwordVariable: 'PASSW', usernameVariable: 'USER')]) {
                    sh 'git config --global user.name ${USER}'
                } //quirks relating to the ability to commit and push later on. "newgithub" is the id of a github username/password credentials stored in Jenkins
                sh 'git checkout ${BRANCH_NAME}' 
                sh 'git pull' //jenkins doesn't seem to pull the whole branch so just in case
                sh 'export AWS_DEFAULT_REGION=$AWS_DEFAULT_REGION' //here and below, setting up the appropriate env variables 
//                                                                     so that the calls to AWS will work                
                sh 'export AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID'
                sh 'export AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY'
                sh 'cloudguard proact -v -i proact/cloudformation/global/cloudguard.yml -t $DOME9TOKEN ' //running the scan
// currently the official release requires cloudguard CLI to run in an environment that has docker. alternatively you can
// add the currently experimental --no-docker flag to the command, which will cause the cli to run usermode udocker.
// for details about the input file (cloudguard.yml) see https://github.com/protegolabs/protego-examples/blob/master/proact/cloudguard/template/cloudguard.yml                
            }
            post{  
                success{ //clouduard proact outputs its results to <workspace>/cloudguard-output folder, and is thus made 
//            available for further human or programmatic scrutiny. Some customers might want to add the output of the scan
//            especially if successful, to the repo itself. here we add it directly to master.
                    sh 'git add .'
                    sh 'git status'
                    sh 'git commit -m jenk-proact-${BUILD_NUMBER}'
		            sh 'git push --set-upstream origin ${BRANCH_NAME}' 
                }
            }
        }
        stage('InsertFSP'){  //here we add a new stage, to add the runtime protection layer to a CFT-deployed function
            steps {
		        sh 'cd $WORKSPACE'
                sh 'git checkout ${BRANCH_NAME}'
            	sh 'git pull'
                sh 'export AWS_DEFAULT_REGION=$AWS_DEFAULT_REGION'
                sh 'export AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID'
                sh 'export AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY'
                sh 'cloudguard fsp add -v -C proact/cloudformation/global/cf.yaml -t $DOME9TOKEN --region $AWS_DEFAULT_REGION'
 //              this the command to add the runtime protection layer (aka Function Self Protection layer) to a function, where 
 //              the function is deployed via a CFT (in this case, it's cf.yaml). the command will create a new CFT template in
 //              the same folder as the original CFT. the name would be <original name>.protected.yaml.
                sh 'git add .' //you may want to be more specific than this
                sh 'git commit -m jenk-FSP-${BUILD_NUMBER}'
    		    sh 'git push --set-upstream origin ${BRANCH_NAME}'
                  
		    }
        }
    }
}
