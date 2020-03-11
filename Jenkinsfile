pipeline {
    // 指定jenkins slave节点(label需与jenkins slave节点配置的label一致)
    agent { label 'jenkins-slave' }
    
    // 定义构建参数
    parameters { 
        string(name: 'DEPLOY_TO_ENV', defaultValue: 'dev', description: 'profileActive 环境选择？')
        string(name: 'DEPLOY_TO_HOST', defaultValue: 'tcp://192.168.0.88:2375', description: '服务部署到哪台机器(82-86)？')       
        booleanParam(name: 'SERVER1_BUILD', defaultValue: true, description: '是否构建wb-strategy？') 
        booleanParam(name: 'SERVER2_BUILD', defaultValue: true, description: '是否构建wb-system？') 
        booleanParam(name: 'SERVER3_BUILD', defaultValue: true, description: '是否构建wb-adapter？') 
        booleanParam(name: 'SERVER4_BUILD', defaultValue: true, description: '是否构建wb-gateway？')        
    }
    
    // 指定流水线执行超时时间
    options {
        timeout(time: 1, unit: 'HOURS') 
    }
    
    // 定义环境变量
    environment { 
               AN_ACCESS_KEY = credentials('harbor') // jenkins凭据,使用密文的方式在后续step中使用
               HARBOR_DOMAIN = 'shinetech-harbor.com' // 镜像仓库地址
               PROJECT = 'dev'  // 镜像仓库的项目名称
               SERVER1 = 'wb-strategy'  // 要构建的第一个服务
               SERVER2 = 'wb-system'
               SERVER3 = 'wb-adapter'
               SERVER4 = 'wb-gateway'
               
            }
            
    // 需与jenkins页面全局工具配置中的名称保持一致,用于后续step调用        
    tools {
        maven 'maven-3.3.9' 
    }
    stages {
        // checkout 代码
        stage ('checkout') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/3.0']], 
                doGenerateSubmoduleConfigurations: false, 
                extensions: [], 
                submoduleCfg: [], 
                userRemoteConfigs: [[credentialsId: 'ef186c77-ca33-4746-b343-d6ff5d450a71', 
                url: 'http://shinetech@192.168.0.72:9091/shinetech/workbench.git']]]) 
            }
        }
        
        // maven 构建
        stage('maven-Build') { 
            steps {
                sh 'mvn clean install -Dmaven.test.skip=true' 
                 
            }
            }
        
        // 生成镜像并推送到镜像仓库,允许保留多个镜像版本,最新的版本指向latest
        stage('Build-SERVER1') { 
            when {
                environment name: 'SERVER1_BUILD', value: 'true'
            }
            steps {               
                dir("${SERVER1}/build"){ 
                    sh 'cp $WORKSPACE/${SERVER1}/target/${SERVER1}-1.0.0.war .'
                    sh 'docker build --force-rm -t ${HARBOR_DOMAIN}/${PROJECT}/${SERVER1}:${BUILD_NUMBER} .'
                    sh 'docker tag ${HARBOR_DOMAIN}/${PROJECT}/${SERVER1}:${BUILD_NUMBER} ${HARBOR_DOMAIN}/${PROJECT}/${SERVER1}:latest'
                    sh 'docker login ${HARBOR_DOMAIN} -u ${AN_ACCESS_KEY_USR} -p ${AN_ACCESS_KEY_PSW}'
                    sh 'docker push ${HARBOR_DOMAIN}/${PROJECT}/${SERVER1}:${BUILD_NUMBER}'
                    sh 'docker push ${HARBOR_DOMAIN}/${PROJECT}/${SERVER1}:latest'
              }
            }
        }
        
                
        stage('Build-SERVER2') { 
            when {
                environment name: 'SERVER2_BUILD', value: 'true'
            }
            steps {
                dir("${SERVER2}/build"){ 
                    sh 'cp $WORKSPACE/${SERVER2}/target/${SERVER2}-1.0.0.war .'
                    sh 'docker build --force-rm -t ${HARBOR_DOMAIN}/${PROJECT}/${SERVER2}:${BUILD_NUMBER} .'
                    sh 'docker tag ${HARBOR_DOMAIN}/${PROJECT}/${SERVER2}:${BUILD_NUMBER} ${HARBOR_DOMAIN}/${PROJECT}/${SERVER2}:latest'
                    sh 'docker login ${HARBOR_DOMAIN} -u ${AN_ACCESS_KEY_USR} -p ${AN_ACCESS_KEY_PSW}'
                    sh 'docker push ${HARBOR_DOMAIN}/${PROJECT}/${SERVER2}:${BUILD_NUMBER}'
                    sh 'docker push ${HARBOR_DOMAIN}/${PROJECT}/${SERVER2}:latest'
              }
            }
        }
        
                      
      stage('Build-SERVER3') { 
            when {
                environment name: 'SERVER3_BUILD', value: 'true'
            }
            steps {
                dir("${SERVER3}/build"){ 
                    sh 'cp $WORKSPACE/${SERVER3}/target/${SERVER3}-1.0.0.war .'
                    sh 'docker build --force-rm -t ${HARBOR_DOMAIN}/${PROJECT}/${SERVER3}:${BUILD_NUMBER} .'
                    sh 'docker tag ${HARBOR_DOMAIN}/${PROJECT}/${SERVER3}:${BUILD_NUMBER} ${HARBOR_DOMAIN}/${PROJECT}/${SERVER3}:latest'
                    sh 'docker login ${HARBOR_DOMAIN} -u ${AN_ACCESS_KEY_USR} -p ${AN_ACCESS_KEY_PSW}'
                    sh 'docker push ${HARBOR_DOMAIN}/${PROJECT}/${SERVER3}:${BUILD_NUMBER}'
                    sh 'docker push ${HARBOR_DOMAIN}/${PROJECT}/${SERVER3}:latest'
              }
            }
        }
        
        
       stage('Build-SERVER4') { 
            when {
                environment name: 'SERVER4_BUILD', value: 'true'
            }
            steps {
                dir("${SERVER4}/build"){ 
                    sh 'cp $WORKSPACE/${SERVER4}/target/${SERVER4}-1.0.0.war .'
                    sh 'docker build --force-rm -t ${HARBOR_DOMAIN}/${PROJECT}/${SERVER4}:${BUILD_NUMBER} .'
                    sh 'docker tag ${HARBOR_DOMAIN}/${PROJECT}/${SERVER4}:${BUILD_NUMBER} ${HARBOR_DOMAIN}/${PROJECT}/${SERVER4}:latest'
                    sh 'docker login ${HARBOR_DOMAIN} -u ${AN_ACCESS_KEY_USR} -p ${AN_ACCESS_KEY_PSW}'
                    sh 'docker push ${HARBOR_DOMAIN}/${PROJECT}/${SERVER4}:${BUILD_NUMBER}'
                    sh 'docker push ${HARBOR_DOMAIN}/${PROJECT}/${SERVER4}:latest'
              }
            }
        }
        
        // 部署服务
        stage('Deploy-SERVER1') { 
            when {
                environment name: 'SERVER1_BUILD', value: 'true'
            }
            steps {
                dir("${SERVER1}/build"){ 
                    sh '''sed -i 's/^DEPLOY_ENV=.*$/DEPLOY_ENV='$DEPLOY_TO_ENV'/' .env'''
                    sh 'docker-compose -H ${DEPLOY_TO_HOST} up -d'
              }
            }
        }
        
        
        stage('Deploy-SERVER2') { 
            when {
                environment name: 'SERVER2_BUILD', value: 'true'
            }
            steps {
                dir("${SERVER2}/build"){ 
                    sh '''sed -i 's/^DEPLOY_ENV=.*$/DEPLOY_ENV='$DEPLOY_TO_ENV'/' .env'''
                    sh 'docker-compose -H ${DEPLOY_TO_HOST} up -d'
              }
            }
        }
        
        
        stage('Deploy-SERVER3') { 
            when {
                environment name: 'SERVER3_BUILD', value: 'true'
            }
            steps {
                dir("${SERVER3}/build"){ 
                    sh '''sed -i 's/^DEPLOY_ENV=.*$/DEPLOY_ENV='$DEPLOY_TO_ENV'/' .env'''
                    sh 'docker-compose -H ${DEPLOY_TO_HOST} up -d'
              }
            }
        }
        
        
        stage('Deploy-SERVER4') { 
            when {
                environment name: 'SERVER4_BUILD', value: 'true'
            }
            steps {
                dir("${SERVER4}/build"){ 
                    sh '''sed -i 's/^DEPLOY_ENV=.*$/DEPLOY_ENV='$DEPLOY_TO_ENV'/' .env'''
                    sh 'docker-compose -H ${DEPLOY_TO_HOST} up -d'
              }
            }
        }
                   
        }
    }
