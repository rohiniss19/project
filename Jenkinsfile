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
    Country = "INDIA"
    No_of_holidays = '5'
    Not_holiday_today = true
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
                  
                // Getting current date    
                  Date date = new Date()
                  today_date=date.format('yyyy-MM-dd')
                  echo "${today_date}"
                  
                // Making rest API call to get the response from the below URL
               
                 /*
                  def response = httpRequest authentication: 'Rohini', url:'http://localhost:8080/api/json?pretty=true'
                  println("Status: "+response.status)
                  println("Content: "+response.content)
                  
                  */
                  
                // Please note that the URL provided for making rest api call is not working. Hence the stage 2 is not working as expected. 
                //Instead I have added a extra holiday.json file with random dates. 
                // The stage 2 checks the holiday.json file to get the holiday dates and matches with current date.
                //If it matches then the build proceed
                 
                    
                 // Reading the holiday.json file to get list of holiday   
                  def holidays = readJSON file: "${env.WORKSPACE}/holiday.json"
                 
                 // Checking if current date is in holiday.json    
                  for (int i=1; i <= 5; i++){
                  echo holidays."""$Country"""."""Date$i"""                                       
                  if (holidays."""$Country"""."""Date$i""" == today_date){
                      Not_holiday_today = false
                      break                      
                  }                  
                  }
                  
                  if (Not_holiday_today)
                  {
                      echo "Today is a regular work day, Hence proceeding with further steps"
                  }
                  else
                  {
                      echo "Today is a holiday, hence skipping all the other steps"
                  }
                  
                  }
                
                }
            }
         
          stage('Build') {
            steps {
                script{
                // Executing stage only if date is not a holiday
                  if (Not_holiday_today)
                  {
                // Creating Build directory      
                   bat """md Builds"""
                   
                // Reading Build.json file
                   def config = readJSON file: "${env.WORKSPACE}/Build.json"
                   
               // Creating Artifacts for each stage
                 Zipping ("Stage1",config.Stage1.Name,config.Stage1.Content)
                 Zipping ("Stage2",config.Stage2.Name,config.Stage2.Content)
                 Zipping ("Stage3",config.Stage3.Name,config.Stage3.Content)
                 Zipping ("Stage4",config.Stage4.Name,config.Stage4.Content)
                 
                //  Creating a Build.zip folder 
                 bat """ tar.exe -a -c -f Build.zip Builds """
 
                  }
                }
               
            }
          }
          stage('Parallel Stage') {
              // Implementing parallel builds - Unit Test stage is run parallel to QA and static stage
              // Static_Check and QA Stages are nested so that they run parallel with unit test stage
            parallel {
             stage('Static Check & QA') {
                    stages{
                         stage('Static Check') {
                    
            steps {
               
                 script{
                 if (Not_holiday_today)
                  {
                       if (params.Static_Check) {
                            // Stage runs only if Current date is not a holiday and corresponding stage is enabled
                             echo "Excecuting static check"
                             Static_check_Run = true
                             copy_files ("Static_Check")
                                          }
                  }
                }
               
            }
            
          }
          
          stage('QA') {
            steps {
                 script{
                 if (Not_holiday_today)
                  {
                       if (params.QA) {
                            // Stage runs only if Current date is not a holiday and corresponding stage is enabled
                            echo "Excecuting QA"
                           QA_Run = true
                           copy_files("QA")
                          
                       }
                  }
                  
                }
               
            }
            
          }
                    }
          }
          stage('Unit Tests') {
            steps {
                 script{
                 if (Not_holiday_today)
                  {
                       if (params.Unit_Test) {
                          // Stage runs only if Current date is not a holiday and corresponding stage is enabled
                           echo "Excecuting Unit Test"
                           Unit_Test_Run = true
                           copy_files ("Unit_Test")
                           
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
                   if (Not_holiday_today){
                   
                 // Printing the summary to the output console regarding the stages ran  
                 echo "Summary"
                 if (Static_check_Run == true){
                    println("Static Tests were executed and corresponding files were copied")
                }
                 if (QA_Run == true){
                    println("QA Tests were executed and corresponding files were copied")
                }
                
                if (Unit_Test_Run == true){
                    println("Unit Tests were executed and corresponding files were copied")
                }
               
                }
               }          
            }
            
            
            post {
               
                failure {
                    
                    // Email sent to corresponding recipents
                    emailext body: 'Check console output at $BUILD_URL to view the results. \n\n ${CHANGES} \n\n -------------------------------------------------- \n${BUILD_LOG, maxLines=100, escapeHtml=false}', 
                   
                    to: "${Failure_Email}", 
                    subject: 'Build failed in Jenkins: $PROJECT_NAME - #$BUILD_NUMBER'
        }
        success {
                    // Email sent to corresponding recipents
                    emailext body: 'Check console output at $BUILD_URL to view the results. \n\n ${CHANGES} \n\n -------------------------------------------------- \n${BUILD_LOG, maxLines=100, escapeHtml=false}', 
                
                    to: "${Success_Email}", 
                    subject: 'Successful build in Jenkins: $PROJECT_NAME - #$BUILD_NUMBER'
        }
        
            }
        }
    }
}



def Zipping (Stage,Name_Stage,Content_Stage)
{
 
  //Creating required zip files
  writeFile(file: """${Name_Stage}.txt""", text: """${Content_Stage}""")
  bat """ move ${Name_Stage}.txt Builds/${Name_Stage}.txt """
 
}

def copy_files (Stage_name)
{
    // Copying required files 
   bat """ md ${Stage_name} """
   bat """ copy  "Builds\\${Stage_name}.txt" "${Stage_name}\\${Stage_name}.txt" """
  
}
   


