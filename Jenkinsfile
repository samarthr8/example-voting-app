pipeline{
//    agent any
    agent {label 'worker'}
    options{
        buildDiscarder(logRotator(daysToKeepStr: '7'))
        disableConcurrentBuilds()
        timeout(time: 30, unit: 'MINUTES')
        retry(3)
    }
    parameters{
        string(name: 'BRANCH', defaultValue: 'master')
        booleanParam(name: 'UnitTestCases', defaultValue: false)
        choice(name: 'Environment', choices: ['Dev', 'QA', 'Uat', 'Prod'])
    }
    stages{
        stage("First Stage:move to vote directory"){
            steps{
                sh "echo Hello!"
                sh "echo Building ${BRANCH} branch!"
                sh "cd vote"
            }
        }
        stage("Second Stage: ecr"){
            steps{
                sh "aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 609042131998.dkr.ecr.us-east-1.amazonaws.com"
                sh "docker build -t 609042131998.dkr.ecr.us-east-1.amazonaws.com/jenkins-setup:v${BUILD_NUMBER} ."
                sh "docker push 609042131998.dkr.ecr.us-east-1.amazonaws.com/jenkins-setup:v${BUILD_NUMBER}"
            }
        }
        stage("Third Stage: ECR image"){
            steps{
                sh "ECR_IMAGE="609042131998.dkr.ecr.us-east-1.amazonaws.com/jenkins-setup:v${BUILD_NUMBER}""
            }
        }
        stage("Fourth Stage: Task Definition"){
            steps{
                sh "TASK_DEFINITION=$(aws ecs describe-task-definition --task-definition vote-c39-fargate:2 --region us-east-1)"
                sh "NEW_TASK_DEFINTIION=$(echo $TASK_DEFINITION | jq --arg IMAGE "$ECR_IMAGE" '.taskDefinition | .containerDefinitions[0].image = $IMAGE | del(.taskDefinitionArn) | del(.revision) | del(.status) | del(.requiresAttributes) | del(.compatibilities) | del(.registeredAt) | del(.registeredBy)')"
                sh "NEW_TASK_INFO=$(aws ecs register-task-definition --region us-east-1 --cli-input-json "$NEW_TASK_DEFINTIION")"
                sh "NEW_REVISION=$(echo $NEW_TASK_INFO | jq '.taskDefinition.revision')"
            }
        }    
        stage("Fifth Stage: Task def update"){
            steps{
                sh "aws ecs update-service --cluster jenkins-vote  --service vote-jk --region us-east-1 --task-definition vote-c39-fargate:${NEW_REVISION}"
            }
        }    
    }    
    
    post { 
        always { 
            echo 'I will always say Hello again!'
        }
        failure{
            echo ' I will run on failure'	
        }
    }
}
