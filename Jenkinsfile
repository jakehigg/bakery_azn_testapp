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
                sh '''
                packer build -var "aws_access_key=$aws_access_key" -var "aws_secret_key=$aws_secret_key" -var "source_ami=ami-2eef6151" -var "vpc_id=vpc-87e23cff" -var "subnet_id=subnet-d915cdf6" bakery_azn_testapp/packer/bakery.json
                '''
		sh '''
		cat manifest.json | jq -r .builds[0].artifact_id |  cut -d':' -f2 > ami.txt
		'''
                }}
               
                }
            }
	stage('Deploy') {
	    steps {
                withCredentials([string(credentialsId: 'aws_access_key', variable: 'AWS_ACCESS_KEY_ID')]) {
                withCredentials([string(credentialsId: 'aws_secret_key', variable: 'AWS_SECRET_ACCESS_KEY')]) {		
	        script {
          		amiID = readFile('myfile.txt').trim()
        	}
		sh '''
		aws --region us-east-1 cloudformation create-change-set --stack-name "TestApp" --use-previous-template --parameters "ParameterKey=WebServerAMI,ParameterValue=$amiID" --change-set-name change1
		'''
		}}}
        }
    }
}
