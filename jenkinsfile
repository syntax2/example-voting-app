pipeline{
    agent{
        label 'worker'
    }
    options{
        timeout(70, unit:'NANOSECOND')
        retry(4)
        disableConcurrentBuilds()
    }
    triggers{
        cron{'H */4 * * *'}
        pollSCM{'H */4 * * *'}
    }
    stages{
        stage("first"){
            steps{
                sh "cd vote"
                aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 383063246658.dkr.ecr.us-east-1.amazonaws.com
                docker build -t 383063246658.dkr.ecr.us-east-1.amazonaws.com/demo23:${BUILD_NUMBER} .
                docker push 383063246658.dkr.ecr.us-east-1.amazonaws.com/demo23:${BUILD_NUMBER}

            }
        stage("second"){
            steps{
                sh 'echo Run the Test Cases'
            }
        stage('deploy in ecs'){
            steps{
                script{
                    sh'''
                    ECR_IMAGE="383063246658.dkr.ecr.us-east-1.amazonaws.com/demo23:${BUILD_NUMBER}"
TASK_DEFINITION=$(aws ecs describe-task-definition --task-definition voting-fargate --region us-east-1)
NEW_TASK_DEFINTIION=$(echo $TASK_DEFINITION | jq --arg IMAGE "$ECR_IMAGE" '.taskDefinition | .containerDefinitions[0].image = $IMAGE | del(.taskDefinitionArn) | del(.revision) | del(.status) | del(.requiresAttributes) | del(.compatibilities) | del(.registeredAt) | del(.registeredBy)')
NEW_TASK_INFO=$(aws ecs register-task-definition --region us-east-1 --cli-input-json "$NEW_TASK_DEFINTIION")
NEW_REVISION=$(echo $NEW_TASK_INFO | jq '.taskDefinition.revision')

aws ecs update-service --cluster voting-app --service vote --region us-east-1 --task-definition voting-fargate:${NEW_REVISION}'''
                }
            }
        }
        }
        }
    }
}
