pipeline {
	agent any
	environment {
		appIP="35.246.109.41";
		gitRepo="https://github.com/amit022017/SpringBoot-Jenkins.git";
		repoName="SpringBoot-Jenkins";
		databaseIP="34.105.179.211";
	}
	stages{
		stage('Test Application'){
			steps{
			sh 'mvn clean test'
			}
		}
		stage('Save Tests'){
			steps{
			sh 'mkdir -p /home/jenkins/Tests/${BUILD_NUMBER}_tests/'
			sh 'mv ./target/surefire-reports/*.txt /home/jenkins/Tests/${BUILD_NUMBER}_tests/'
			}
		}
		stage('SSH Build Deploy'){
			steps{
			sh '''ssh -i "~/.ssh/jenkins_key" jenkins@$appIP << EOF
			rm -rf $repoName
			git clone $gitRepo
			cd $repoName
			rm -f ./src/main/resources/application-dev.properties
			echo 'spring.jpa.hibernate.ddl-auto=create-drop
spring.h2.console.enabled=false
spring.h2.console.path=/h2

spring.datasource.url=jdbc:mysql://$databaseIP:3306/tdl
spring.datasource.data=classpath:data-dev.sql
spring.datasource.username=root
spring.datasource.password=secret' > ./src/main/resources/application-dev.properties
			mvn clean package
			'''
			}
		}
		stage('Moving War'){
			steps{
			sh '''ssh -i "~/.ssh/jenkins_key" jenkins@$appIP	 << EOF
			cd $repoName
			mkdir -p /home/jenkins/Wars
			mv ./target/*.war /home/jenkins/Wars/project_war.war
			'''
			}
                }
		stage('Stopping Service'){
			steps{
			sh '''ssh -i "~/.ssh/jenkins_key" jenkins@$appIP << EOF
			cd $repoName
			bash stopservice.sh
			'''
			}
		}
		stage('Create new service file'){
			steps{
			sh '''ssh -i "~/.ssh/jenkins_key" jenkins@$appIP << EOF
			mkdir -p /home/jenkins/appservice
			echo '#!/bin/bash
sudo java -jar /home/jenkins/Wars/project_war.war' > /home/jenkins/appservice/start.sh
sudo chmod +x /home/jenkins/appservice/start.sh
echo '[Unit]
Description=My SpringBoot App

[Service]
User=ubuntu
Type=simple

ExecStart=/home/jenkins/appservice/start.sh

[Install]
WantedBy=multi-user.target' > /home/jenkins/myApp.service
sudo mv /home/jenkins/myApp.service /etc/systemd/system/myApp.service
			'''
			}
		}
		stage('Reload and restart service'){
			steps{
			sh '''ssh -i "~/.ssh/jenkins_key" jenkins@$appIP << EOF
			sudo systemctl daemon-reload
			sudo systemctl restart myApp
			'''
			}
		}

	}
}
