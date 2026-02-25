pipeline {

    agent any

    options {
        skipStagesAfterUnstable()
    }

    stages {

        stage('Get Code') {
    steps {
        deleteDir()

        sh '''
        whoami
		hostname
        '''

        git branch: 'master', url: 'https://github.com/AlfredoVG77/todo-list-aws.git'

        stash name: 'code', includes: '**/*'
    }
}

	
			stage('Deploy to Production') {
				steps {
					catchError(buildResult: 'FAILURE', stageResult: 'FAILURE') {

					unstash 'code'

					sh '''
							echo '=== SAM BUILD ==='
							sam build --template-file template.yaml
							sam validate --region us-east-1

							echo '=== SAM DEPLOY TO STAGING ==='
							sam deploy --stack-name todo-list-aws-production --region us-east-1 --capabilities CAPABILITY_IAM --resolve-s3 --s3-prefix todo-list-aws --parameter-overrides Stage=production --no-fail-on-empty-changeset --no-confirm-changeset || true
						'''
					}
				}
			}
			stage('API Tests') {
    steps {

        unstash 'code'

        sh '''
            whoami
			hostname
			echo "=== Obteniendo URL de la API desde CloudFormation ==="
            BASE_URL=$(aws cloudformation describe-stacks --stack-name todo-list-aws-production --query "Stacks[0].Outputs[?OutputKey=='BaseUrlApi'].OutputValue" --region us-east-1 --output text)

            echo "=== Ejecutando tests de integración ==="

                . /var/lib/jenkins/venv/bin/activate
                export BASE_URL="$BASE_URL"
                pytest test/integration/todoApiTest.py -k 'test_api_listtodos or test_api_gettodo' --junitxml=api-tests.xml -q --disable-warnings --maxfail=1
            '''
            junit 'api-tests.xml'
          }
        }
    

	}
}
