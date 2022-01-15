// Importing required libraries
import groovy.json.JsonOutput
import java.text.SimpleDateFormat
import groovy.json.*
import java.io.*
import net.sf.json.groovy.JsonSlurper
import groovy.text.StreamingTemplateEngine;

pipeline {
    agent any
    
    environment {
    // Environment Variables    
    today_date = ""
    holiday_date = ""
    QA_Run = false
    Static_check_Run = false
    Unit_Test_Run = false
    name = ""
   }
    
    parameters {
        // Input parameters
         booleanParam(defaultValue: true,name: 'Static_Check', description: 'Static_Check?')
         booleanParam(defaultValue: true,name: 'QA', description: 'QA?')
         booleanParam(defaultValue: true,name: 'Unit_Test', description: 'Unit_Test?')
         string(defaultValue: 'rohinissankannavar@gmail.com',name: 'Success_Email', description: 'Please tell me whom to send success mail?')
         string(defaultValue: 'ssankannavarrohini@gmail.com',name: 'Failure_Email', description: 'Please tell me whom to send Failure mail?')
     
    }

    stages {
        stage('Git pull stage') {
            steps {
               
               //cleaning workspace before build
                cleanWs()
                 // Get some code from a GitHub repository
                checkout([$class: 'GitSCM',
                    branches: [[name: "master"]],
                    doGenerateSubmoduleConfigurations: false,
                    extensions: [],
                    submoduleCfg: [],
                    userRemoteConfigs: [[
                        credentialsId: '',
                        url: 'https://github.com/rohiniss19/project.git']]])

            }
        }
        stage('Is the run required?') {
            steps {
                
                script{
                  Date date = new Date()
                  today_date=date.format('yyyy-MM-dd')
                  echo "${today_date}"
                  holiday_date= "2022-01-15"
                  echo "${holiday_date}"
                  if (today_date == holiday_date)
                  {
                      echo "heyy"
                  }
                  
               
                  }
                
                }
            }
         
          stage('Build') {
            steps {
                script{
                
                 if (today_date == holiday_date )
                  {
                     
                   bat """md Builds"""
                   def config = readJSON file: "${env.WORKSPACE}/Build.json"
               
                 Zipping ("Stage1",config.Stage1.Name,config.Stage1.Content)
                 Zipping ("Stage2",config.Stage2.Name,config.Stage2.Content)
                 Zipping ("Stage3",config.Stage3.Name,config.Stage3.Content)
                 
                 bat """ tar.exe -a -c -f Build.zip Builds """
                 echo " Build is running"
 

                      
                  }
                }
               
            }
          }
          stage('Parallel Stage') {
            parallel {
             stage('Static Check & QA') {
                    stages{
                         stage('Static Check') {
                    
            steps {
                // Get some code from a GitHub repository
                 script{
                 if (today_date == holiday_date)
                  {
                       if (params.Static_Check) {
                           Static_check_Run = true
                      echo "Excecuting static check"
                     copy_files ("Static_Check")
                                          }
                  }
                }
               
            }
            
          }
          
          stage('QA') {
            steps {
                // Get some code from a GitHub repository
                 script{
                 if (today_date == holiday_date  )
                 
                  {
                       if (params.QA) {
                           QA_Run = true
                           copy_files("QA")
                           echo "Excecuting QA"
                       }
                  }
                  
                }
               
            }
            
          }
                    }
          }
          stage('Unit Tests') {
            steps {
                // Get some code from a GitHub repository
                 script{
                 if (today_date == holiday_date  )
                 
                  {
                       if (params.Unit_Test) {
                           
                           Unit_Test_Run = true
                           copy_files ("Build")
                           echo "Excecuting Unit Test"
                       }
                  }
                }
               
            }
            
          }
            }
          }
          
            stage('Summary') {
            steps {
               script{
                // Get some code from a GitHub repository
                 if (Static_check_Run == true){
                    println("Static Tests were executed and corresponding files were copied")
                }
                 if (QA_Run == true){
                    println("QA Tests were executed and corresponding files were copied")
                }
                
                if (Unit_Test_Run == true){
                    println("Unit Tests were executed and corresponding files were copied")
                }
                echo "Summary"
                }
            }
            
            
            post {
               
                failure {
            emailext body: 'Check console output at $BUILD_URL to view the results. \n\n ${CHANGES} \n\n -------------------------------------------------- \n${BUILD_LOG, maxLines=100, escapeHtml=false}', 
                   
                    to: "${Failure_Email}", 
                    subject: 'Build failed in Jenkins: $PROJECT_NAME - #$BUILD_NUMBER'
        }
        success {
            emailext body: 'Check console output at $BUILD_URL to view the results. \n\n ${CHANGES} \n\n -------------------------------------------------- \n${BUILD_LOG, maxLines=100, escapeHtml=false}', 
                
                    to: "${Success_Email}", 
                    subject: 'Unstable build in Jenkins: $PROJECT_NAME - #$BUILD_NUMBER'
        }
        
            }
        }
    }
}



def Zipping (Stage,Name_Stage,Content_Stage)
{
 
 echo "${Name_Stage}"
 echo "${Content_Stage}"

  writeFile(file: """${Name_Stage}.txt""", text: """${Content_Stage}""")
  bat """ move ${Name_Stage}.txt Builds/${Name_Stage}.txt """
 
}

def copy_files (Stage_name)
{
   bat """ md ${Stage_name} """
   bat """ copy  "Builds\\${Stage_name}.txt" "${Stage_name}\\${Stage_name}.txt" """
  
}
   



