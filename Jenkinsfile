pipeline {
    agent any 

    stages {
        stage('Git Packer Template') { 
            steps { 
                sh 'rm -rvf bakery_azn_testapp'
                sh 'git clone https://github.com/jakehigg/bakery_azn_testapp.git' 
            }
        }
        stage('Bake') {
            steps {
                withCredentials([string(credentialsId: 'aws_access_key', variable: 'aws_access_key')]) {
                withCredentials([string(credentialsId: 'aws_secret_key', variable: 'aws_secret_key')]) {
                withCredentials([string(credentialsId: 'aws_access_key', variable: 'AWS_ACCESS_KEY_ID')]) {
                withCredentials([string(credentialsId: 'aws_secret_key', variable: 'AWS_SECRET_ACCESS_KEY')]) {
                sh '''
		rm -f manifest.json
		export latestgold="$(aws --region us-east-1 ssm get-parameter --name "latestgold" | jq -r  .Parameter.Value)"
                packer build -var "aws_access_key=$aws_access_key" -var "aws_secret_key=$aws_secret_key" -var "source_ami=$latestgold" -var "vpc_id=vpc-87e23cff" -var "subnet_id=subnet-d915cdf6" bakery_azn_testapp/packer/bakery.json
                export newapp=$(cat manifest.json | jq -r .builds[0].artifact_id |  cut -d":" -f2)
                aws --region us-east-1 ssm put-parameter --name "latestapp" --value "$newapp" --type "String" --overwrite		
		'''
                }}}}
               
                }
            }
	stage('Create Change Set') {
	    steps {
                withCredentials([string(credentialsId: 'aws_access_key', variable: 'AWS_ACCESS_KEY_ID')]) {
                withCredentials([string(credentialsId: 'aws_secret_key', variable: 'AWS_SECRET_ACCESS_KEY')]) {		
		sh '''
		export amiID=$(aws --region us-east-1 ssm get-parameter --name "latestapp" | jq -r  .Parameter.Value)
		aws --region us-east-1 cloudformation create-change-set --stack-name "TestApp2" --template-body file://$PWD/bakery_azn_testapp/CFN.json --parameters ParameterKey="WebServerAMI",ParameterValue="$amiID" ParameterKey="KeyName",UsePreviousValue=true ParameterKey="Subnets",UsePreviousValue=true ParameterKey="VpcId",UsePreviousValue=true ParameterKey="ALBSubnets",UsePreviousValue=true --change-set-name $amiID
		'''
		}}}
        }
	stage('Deployment Approval') {
            steps {
	    input "Deploy Change Set?"
	}
	}
	stage('Deploy to QA') {
            steps {
                withCredentials([string(credentialsId: 'aws_access_key', variable: 'AWS_ACCESS_KEY_ID')]) {
                withCredentials([string(credentialsId: 'aws_secret_key', variable: 'AWS_SECRET_ACCESS_KEY')]) {
                sh '''
                export amiID=$(aws --region us-east-1 ssm get-parameter --name "latestapp" | jq -r  .Parameter.Value)
                aws --region us-east-1 cloudformation execute-change-set --stack-name "grabthegrapes1" --change-set-name $amiID
                '''
                }}}
        }
    }
}
