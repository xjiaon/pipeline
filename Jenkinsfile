pipeline {
    agent any
    options{ 
    	timeout(time: 1,unit: 'HOURS')//��ʱ����
    	retry(2)//���Դ���
    	buildDiscarder(logRotator(numToKeepStr: '10'))//���ֹ�����������
    }
    triggers { //��������ƻ�
    	pollSCM('10 * * * *') 
    }
    tools{ 
        maven 'Maven3.5'
        jdk 'java8'
    }
    stages {
        stage('download-code') {
        	//1.����Դ��
            parallel {
                stage('download aliyun') {
                    agent {
                        label "docker-manager-26"
                    }
                    steps {
                    	dir('/home/pipline/build'){
           					checkout([$class: 'SubversionSCM', additionalCredentials: [], excludedCommitMessages: '', excludedRegions: '', excludedRevprop: '', 
           					excludedUsers: '', filterChangelog: false, ignoreDirPropChanges: false, includedRegions: '', locations: [[credentialsId: '6d59f39f-ce88-414e-b0a9-8d78ec0af90e', 
           					depthOption: 'infinity', ignoreExternalsOption: true, remote: 'svn://172.16.1.160/aliyun/aliyun']], workspaceUpdater: [$class: 'UpdateUpdater']])
       					}
                    }
                }
                stage('download sharebuy') {
                    agent {
                        label "docker-manager-26"
                    }
                    steps {
                    	dir('/home/pipline/build'){ 
                    		checkout([$class: 'SubversionSCM', additionalCredentials: [], excludedCommitMessages: '', excludedRegions: '', excludedRevprop: '', 
        					excludedUsers: '', filterChangelog: false, ignoreDirPropChanges: false, includedRegions: '', locations: [[credentialsId: '6d59f39f-ce88-414e-b0a9-8d78ec0af90e', 
        					depthOption: 'infinity', ignoreExternalsOption: true,remote: 'svn://172.16.1.160/sharebuy/sharebuy']], workspaceUpdater: [$class: 'UpdateUpdater']])
                    	}
        				
                    }
                }
            }
        }
        //2.����Դ��aliyun
        stage('mvn aliyun-code') {
        	agent {
            	label "docker-manager-26"
            }
            steps{
            	dir('/home/pipline/build/aliyun'){
           			sh '/home/maven/apache-maven-3.5.0/bin/mvn install'
       			}
            }
        }
    	//3.����-ɨ�����        
        stage('mvn-sonar sharebuy'){ 
    		parallel { 
    			stage('mav sharebuy'){ 
    			//3.1����sharebuy����
    				agent{ 
    					label "docker-manager-26"
    				}
    				steps{ 				 
    					dir('/home/pipline/build/sharebuy'){
        					sh '/home/maven/apache-maven-3.5.0/bin/mvn install'
     					}
    				}
    			}
    			//3.2ɨ�����
    			stage('sonar-sharebuy'){ 
    				agent{ 
    					label "docker-manager-26"
    				}
    				steps{ 
    					dir('/home/pipline/build/sonar'){
        					checkout([$class: 'SubversionSCM', additionalCredentials: [], excludedCommitMessages: '', excludedRegions: '', excludedRevprop: '', excludedUsers: '', 
        					filterChangelog: false, ignoreDirPropChanges: false, includedRegions: '', locations: [[credentialsId: '6d59f39f-ce88-414e-b0a9-8d78ec0af90e', 
        					depthOption: 'infinity', ignoreExternalsOption: true,remote: 'svn://172.16.1.160/sharebuy/sharebuy']], workspaceUpdater: [$class: 'UpdateUpdater']])
    					}
    					dir('/home/pipline/build/sonar/sharebuy'){ 
							sh '/usr/local/sonar-scanner-3.0.3.778/bin/sonar-scanner'
						}   					
    				}
    			}    		
    		}
    	}
    	//4.���س���
    	stage('down-sharebuy') {
        	agent {
            	label "docker-manager-26"
            }
            steps{
            	dir('/home/pipline/build/sharebuy/target'){ 
					sh 'cp -rf sharebuy-1.0-SNAPSHOT /home/pipline/build/src/webapps/sharebuy'
					sh 'docker stop xxbmm-shop'
				}
            }
        }
    	//5.��������
    	stage('build-sharebuy') {
        	agent {
            	label "master"
            }
            steps{
            	dir('/home/pipline/build/docker-build'){ 
					ansiblePlaybook credentialsId: '2688f527-0f4b-453a-af43-6a2aefc56a5a', 
					installation: 'ansible', 
					inventory: '/etc/ansible/hosts', 
					playbook: '/home/ansible/projects/xxbmm/pipline-build-xxbmm.yaml', 
					sudoUser: null
				}
            }
        }
    	//6.�鿴״̬
    	stage('check-webstat') {
        	agent {
            	label "docker-manager-26"
            }
            steps{
            	sh 'docker ps -a'
            }
        }
        
    }
    post{
    	//7.���͹����ʼ� 
    	always{ 
    		emailext body: '${JELLY_SCRIPT,template="templates"}', 
			recipientProviders: [[$class: 'DevelopersRecipientProvider']], 
			subject: '$DEFAULT_SUBJECT', 
			to: 'xiaojian@xxbmm.com'
    	}
    }
 }   