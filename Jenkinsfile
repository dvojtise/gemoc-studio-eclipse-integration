// Pipeline using declarative syntax
// you can fall down to script syntax using the script { } command 
pipeline {
	agent any
	options {
		skipDefaultCheckout(true) // avoid default checkout implied by declarative pipeline (we do it in the prepare stage)
		buildDiscarder( logRotator(numToKeepStr:'60', artifactNumToKeepStr: '1'))
		disableConcurrentBuilds()
	}
	tools {
        	maven 'apache-maven-latest'
        	jdk 'jdk1.8.0-latest'
    }
	environment {
		DOWNLOAD_FOLDER = "/home/data/httpd/download.eclipse.org/gemoc"
	}
	stages {
		
		stage('Prepare') {
			steps {
				lock('myResource') {	
					echo 'Content of the workspace before Checkout'
					sh "ls -lsa"
					// Wipe the workspace so we are building completely clean
	  		  		deleteDir()
					//cleanWs()
					
					// Get code from GitHub repositories				
					// this will check if there is a branch with the same name as the current branch (ie. the branch containing this Jenkinsfile) and use that for the checkout, but if there is no
					// branch with the same name it will fall back to the master branch
					dir('gemoc-studio') {
	    					//	checkout resolveScm(source: git('https://github.com/eclipse/gemoc-studio.git'), targets: [BRANCH_NAME,'master'])
				    		script {
			 				def gemocstudioScm = resolveScm source: [$class: 'GitSCMSource', credentialsId: '', id: '_', remote: 'https://github.com/eclipse/gemoc-studio.git', traits: [[$class: 'BranchDiscoveryTrait'], [$class: 'LocalBranchTrait']]], targets: [BRANCH_NAME, 'master']
			 				checkout gemocstudioScm
			 			}
				    	}
					dir('gemoc-studio-modeldebugging') {
	    					script {
			 				def gemocstudiomodeldebuggingScm = resolveScm source: [$class: 'GitSCMSource', credentialsId: '', id: '_', remote: 'https://github.com/eclipse/gemoc-studio-modeldebugging.git', traits: [[$class: 'BranchDiscoveryTrait'], [$class: 'LocalBranchTrait']]], targets: [BRANCH_NAME, 'master']
			 				checkout gemocstudiomodeldebuggingScm
			 			}
					}
					dir('gemoc-studio-execution-ale') {
	    					script {
			 				def gemocstudiomodeldebuggingScm = resolveScm source: [$class: 'GitSCMSource', credentialsId: '', id: '_', remote: 'https://github.com/eclipse/gemoc-studio-execution-ale.git', traits: [[$class: 'BranchDiscoveryTrait'], [$class: 'LocalBranchTrait']]], targets: [BRANCH_NAME, 'master']
			 				checkout gemocstudiomodeldebuggingScm
			 			}
					}
					dir('gemoc-studio-moccml') {
	    					script {
			 				def gemocstudiomoccmlScm = resolveScm source: [$class: 'GitSCMSource', credentialsId: '', id: '_', remote: 'https://github.com/eclipse/gemoc-studio-moccml.git', traits: [[$class: 'BranchDiscoveryTrait'], [$class: 'LocalBranchTrait']]], targets: [BRANCH_NAME, 'master']
			 				checkout gemocstudiomoccmlScm
			 			}
					}
					dir('gemoc-studio-execution-moccml') {
	    					script {
			 				def gemocstudioexecutionmoccmlScm = resolveScm source: [$class: 'GitSCMSource', credentialsId: '', id: '_', remote: 'https://github.com/eclipse/gemoc-studio-execution-moccml.git', traits: [[$class: 'BranchDiscoveryTrait'], [$class: 'LocalBranchTrait']]], targets: [BRANCH_NAME, 'master']
			 				checkout gemocstudioexecutionmoccmlScm
			 			}
					}
				    	echo 'Content of the workspace after Checkout'
					sh "ls -lsa"
					sh "chmod 777 ./gemoc-studio/dev_support/jenkins/showGitBranches.sh"
		      			sh "./gemoc-studio/dev_support/jenkins/showGitBranches.sh ."
				} // end of lock	
			}
		}
		stage('Build and verify') {
	    		steps { 
				lock('myResource') { 
					script {
				
						def studioVariant
						if(  env.JENKINS_URL.contains("https://hudson.eclipse.org/gemoc/")){
						studioVariant = "Official build"
						} else {
						studioVariant = "${JENKINS_URL}"
						}
						// Run the maven build with tests  
						withEnv(["STUDIO_VARIANT=${studioVariant}","BRANCH_VARIANT=${BRANCH_NAME}"]){ 
						sh 'printenv'         
						   	dir ('gemoc-studio/dev_support/full_compilation') {
								wrap([$class: 'Xvnc', takeScreenshot: false, useXauthority: true]) {
									sh "mvn -Dmaven.test.failure.ignore \"-Dstudio.variant=${studioVariant}\" -Dbranch.variant=${BRANCH_VARIANT} clean verify --errors "
								}
							}      
						}
				   	}
				} // end of lock
			}
			post {
				success {
					junit '**/target/surefire-reports/TEST-*.xml' 
				}
				always {
					sh "./gemoc-studio/dev_support/jenkins/showGitBranches.sh ."
				}
			}
	 	}
		stage("Archive in Jenkins") {
			steps {
				echo "archive artifact"
				archiveArtifacts '**/target/products/*.zip, **/gemoc-studio/gemoc_studio/releng/org.eclipse.gemoc.gemoc_studio.updatesite/target/repository/**, **/docs/org.eclipse.gemoc.studio.doc/target/publish/**, **/screenshots/**'
			}
		}
		stage('Web upload') {
			when {
        		// skip this stage unless on Master branch
        		branch "master"
			}
			steps {
				echo "Deploy products to download.eclipse.org"
				sh 'rm -rf ${DOWNLOAD_FOLDER}/packages/nightly'
				sh 'mkdir -p ${DOWNLOAD_FOLDER}/packages/nightly'
				sh 'cp gemoc-studio/gemoc_studio/releng/org.eclipse.gemoc.gemoc_studio.product/target/products/*.zip ${DOWNLOAD_FOLDER}/packages/nightly'
		        sh 'cp gemoc-studio/gemoc_studio/releng/org.eclipse.gemoc.gemoc_studio.headless.product/target/products/*.zip ${DOWNLOAD_FOLDER}/packages/nightly'
		        sh 'cp gemoc-studio/gemoc_studio/releng/org.eclipse.gemoc.gemoc_studio.product/target/products/index.html ${DOWNLOAD_FOLDER}/packages/nightly'
		        
		        echo "Deploy updatesite to download.eclipse.org"
				sh 'rm -rf   ${DOWNLOAD_FOLDER}/updates/nightly'
				sh 'mkdir -p ${DOWNLOAD_FOLDER}/updates/nightly'
				sh 'cp -r    gemoc-studio/gemoc_studio/releng/org.eclipse.gemoc.gemoc_studio.product/target/repository/* ${DOWNLOAD_FOLDER}/updates/nightly'
				sh 'zip -R   ${DOWNLOAD_FOLDER}/updates/nightly/repository.zip gemoc-studio/gemoc_studio/releng/org.eclipse.gemoc.gemoc_studio.product/target/repository/*'
				
				echo "Deploy documentation archive to download.eclipse.org"
				sh 'rm -rf   ${DOWNLOAD_FOLDER}/docs/nightly'
				sh 'mkdir -p ${DOWNLOAD_FOLDER}/docs/nightly'
				sh 'cp -r    gemoc-studio/docs/org.eclipse.gemoc.studio.doc/target/publish/webhelp/* ${DOWNLOAD_FOLDER}/docs/nightly'				
				sh 'mkdir -p ${DOWNLOAD_FOLDER}/docs/nightly/archive'
				sh 'zip -R   ${DOWNLOAD_FOLDER}/docs/nightly/archive/studio-docs.zip gemoc-studio/docs/org.eclipse.gemoc.studio.doc/target/publish/webhelp/*'
			}
		}
	}
	post { 
		// send mail to main maintainers for unstable builds of master branch
		// note: should also send mail to culprits but currently 
        //       sendToIndividuals doesn't work due to https://issues.jenkins-ci.org/browse/JENKINS-43202
		always {
			script { 
            	if(  "${BRANCH_NAME}".equals("master")){
            		// note: sendToIndividuals doesn't work due to https://issues.jenkins-ci.org/browse/JENKINS-43202
            		step([$class: 'Mailer', notifyEveryUnstableBuild: true, recipients: "didier.vojtisek@inria.fr, erwan.bousse@tuwien.ac.at", sendToIndividuals: true])
			    }
            }  
		}
        // changed { }
        unstable {
        	echo 'Unstable' 
        }
        failure {
        	echo 'Failure' 
    	}
	}
}

