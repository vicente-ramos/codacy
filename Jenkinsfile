pipeline {
    agent { label 'test-2' }
    // agent any
	libraries {
  		lib('jenkins-shared-lib')
	}

    // environment {
    //     PROJECT_NAME = get_project_name()
    //     PR_BRANCH = ""
    // }

    stages{
        //stage("git checkout"){
        //    steps{
                // git 'https://github.com/mavenishere/codacy'
        //        git 'https://github.com/vicente-ramos/codacy.git'
        //    }
        //}
        stage('Build') {
            steps {
                sh label: 'Debug: Print env vars', script: '''printenv | sort'''
            }
        }        
        stage("info"){
            steps{
                writeFile file: 'info.json', text: create_info()
                
            }          
        }
        stage("Compress Files"){
            steps{
                script {
                    def statusCode = compress_files("time.txt", "README.md", "tests.py", "awesome/*")
                    // if ( statusCode == 200 ) {
                    //     echo "Passed! Status Code is : ${statusCode}"
                    // } else {
                    //     throw new hudson.AbortException("[FAILURE] Status Code is not value : ${statusCode}")
                    // }   
                    echo statusCode
                }
            }          
        }    
        stage("Upload Code coverage to codacy.") {
            steps {
                codacy_coverage("codacy", "vicente-ramos", "ruby", "codacy.json", "509740f1261b4a02b24a6f342bbad154")
            }
        }
        stage('codacy') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'token', variable: 'TOKEN')]) {
                        def grade = codacy_report(TOKEN)
                        if ( grade == "B" ) {
                            echo "Grade is of acceptable value : ${grade}"
                        }else {
                            throw new hudson.AbortException("[FAILURE] Grade is below acceptable value : ${grade}")
                        }
                        // last_log = sh(returnStdout: true, script: "git log -1")
                        // echo "Last log " + last_log
                        // last_author = getGitAuthor()
                        // echo "Last git author was " + last_author
                        // author_email = getAuthorEmailid()
                        // echo "Author email is " + author_email
                        // slack_id = slackUserIdFromEmail("${getAuthorEmailid()}")
                        // echo "Slack id is " + slack_id
                        // commiters_slack_id = getAllCommitersId()
                        // echo "The slak ids for all users are " + commiters_slack_id
                    }
                }
            }
        }
        stage('Codacy Script') {
            steps {
                withCredentials([string(credentialsId: 'codacy_project_token', variable: 'TOKEN')]) {
                    codacy_sh('codacy', 'vicente-ramos', 'python', '$TOKEN', 'coverage.xml')
                }
            }
        }                   
    }
    post {
        always{
            slacknotification(currentBuild.currentResult,'#jenkins',getTestResultsForSlack())
            slacknotificationAllCommiters(currentBuild.currentResult,'#jenkins',getTestResultsForSlack())
        }
    }    
}
