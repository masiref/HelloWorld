pipeline {
	agent any

	// Environment Variables
	environment {
		MAJOR = '1'
		MINOR = '0'

		// Orchestrator Services
		// UAT
		UIPATH_UAT_ORCH_URL = "https://masire-orchestrator.westeurope.cloudapp.azure.com/"
		UIPATH_UAT_ORCH_CREDENTIALS_ID = "orchestrator-admin"
		UIPATH_UAT_ORCH_TENANT_NAME = "Default"
		UIPATH_UAT_ORCH_FOLDER_NAME = "01-UAT"
		UIPATH_UAT_ORCH_TESTSET_NAME = "Dummy Test Set"

		// PROD
		UIPATH_PROD_ORCH_URL = "https://masire-orchestrator.westeurope.cloudapp.azure.com/"
		UIPATH_PROD_ORCH_CREDENTIALS_ID = "orchestrator-admin"
		UIPATH_PROD_ORCH_TENANT_NAME = "Prod"
		UIPATH_PROD_ORCH_FOLDER_NAME = "Shared"
	}

	stages {
		// Printing basic information
		stage('Preparing') {
			steps {
				echo "Jenkins Home ${env.JENKINS_HOME}"
				echo "Jenkins URL ${env.JENKINS_URL}"
				echo "Jenkins Job Number ${env.BUILD_NUMBER}"
				echo "Jenkins Job Name ${env.JOB_NAME}"
				checkout scm
			}
		}

		stage('Build') {
			steps {
				echo "Building with ${WORKSPACE} ..."
				UiPathPack (
					outputPath: "Output\\${env.BUILD_NUMBER}",
					projectJsonPath: "project.json",
					version: [$class: 'ManualVersionEntry', version: "${MAJOR}.${MINOR}.${env.BUILD_NUMBER}"],
					useOrchestrator: false,
					traceLevel: 'None'
				)
			}
		}

		stage('Deploy to UAT') {
			input {
				message "Do you want to proceed for UAT deployment?"
			}
			steps {
				echo "Deploying ${BRANCH_NAME} to UAT"
				UiPathDeploy (
					packagePath: "Output\\${env.BUILD_NUMBER}",
					orchestratorAddress: "${UIPATH_UAT_ORCH_URL}",
					orchestratorTenant: "${UIPATH_UAT_ORCH_TENANT_NAME}",
					folderName: "${UIPATH_UAT_ORCH_FOLDER_NAME}",
					environments: '',
					createProcess: true,
					credentials: [$class: 'UserPassAuthenticationEntry', credentialsId: "${UIPATH_UAT_ORCH_CREDENTIALS_ID}"],
					traceLevel: 'None',
					entryPointPaths: 'Main.xaml'
				)
			}
		}

		stage('Run regression tests') {
			steps {
				echo 'Running regression tests ...'
		        UiPathTest (
					testTarget: [$class: 'TestSetEntry', testSet: "${UIPATH_UAT_ORCH_TESTSET_NAME}"],
					parametersFilePath: '',
					orchestratorAddress: "${UIPATH_UAT_ORCH_URL}",
					orchestratorTenant: "${UIPATH_UAT_ORCH_TENANT_NAME}",
					folderName: "${UIPATH_UAT_ORCH_FOLDER_NAME}",
					timeout: 30000,
					traceLevel: 'None',
					testResultsOutputPath: "result.xml",
					credentials: [$class: 'UserPassAuthenticationEntry', credentialsId: "${UIPATH_UAT_ORCH_CREDENTIALS_ID}"]
				)
				step([$class: 'JUnitResultArchiver', testResults: 'result.xml'])
				if (currentBuild.result == 'UNSTABLE')
					currentBuild.result = 'FAILURE'
			}
		}

		stage('Deploy to Production') {
			input {
				message "Do you want to proceed for PROD deployment?"
			}
			steps {
				echo "Deploying ${BRANCH_NAME} to PROD"
				UiPathDeploy (
					packagePath: "Output\\${env.BUILD_NUMBER}",
					orchestratorAddress: "${UIPATH_PROD_ORCH_URL}",
					orchestratorTenant: "${UIPATH_PROD_ORCH_TENANT_NAME}",
					folderName: "${UIPATH_PROD_ORCH_FOLDER_NAME}",
					environments: '',
					createProcess: true,
					credentials: [$class: 'UserPassAuthenticationEntry', credentialsId: "${UIPATH_PROD_ORCH_CREDENTIALS_ID}"],
					traceLevel: 'None',
					entryPointPaths: 'Main.xaml'
				)
			}
		}
	}

	// Options
	options {
		// Timeout for pipeline
		timeout(time: 80, unit: 'MINUTES')
		skipDefaultCheckout()
	}

	// Post deployment
	post {
		success {
			echo 'Deployment has been completed!'
			// cleanWs()
		}
		failure {
			echo "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.JOB_DISPLAY_URL})"
		}
	}
}
