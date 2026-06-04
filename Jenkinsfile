pipeline {
    agent any
    
    environment {
        // Required for checkout scm (Username/Password format) and pushing back to GitHub
        GITHUB_CREDS = credentials('my-github-creds')
        // Fresh secret text token bound for your flexible API call
        GITHUB_TOKEN = credentials('Jenkins-Github')
    }          
    
    stages {
        stage('Safe Source Checkout') {
            steps {
                echo "Starting safe repository checkout..."
				// Pulls the code from GitHub, locking down your baseline files
                checkout scm
                echo "Checkout complete."
            }
        }

        stage('Verify GitHub Auth & Rate Limit') {
            steps {
                echo "Checking GitHub authentication using Secret Text..."
                echo "------------------------------------------------------------------------"
                
			// Native curl passes headers safely. Quote-chopping \"github\" stops Jenkins URL rewriting.	
            // PowerShell parsing block using the flexible %GITHUB_TOKEN% to do a raw query on the  github API 
                bat "C:\\Windows\\System32\\curl.exe -s -H \"Accept: application/vnd.github.v3+json\" -H \"User-Agent: Jenkins-Pipeline\" -H \"Authorization: token %GITHUB_TOKEN%\" \"https://api.\"github\".com/rate_limit\" | powershell -Command \"\$input | ConvertFrom-Json | ForEach-Object { \$_.resources.PSObject.Properties | ForEach-Object { \$time = [System.DateTimeOffset]::FromUnixTimeSeconds(\$_.Value.reset).LocalDateTime.ToString('yyyy-MM-dd HH:mm:ss'); Write-Output ('Stage: ' + \$_.Name.ToUpper().PadRight(28) + ' | Remaining: ' + \$_.Value.remaining.ToString().PadRight(5) + ' / ' + \$_.Value.limit.ToString().PadRight(6) + ' | Resets At: ' + \$time) } }\""
            }
        }

        stage('Automated Bidirectional Sync') {
            steps {
                echo "Checking for local Jenkinsfile updates to sync back to GitHub..."
                
                // Using your credentials block to map user and password strings safely
                withCredentials([usernamePassword(credentialsId: 'my-github-creds', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_PASS')]) {
                    bat """
                        @echo off
                        :: Configure real production git signature for clean traceability
                        git config user.name "mcorries"
                        git config user.email "mcorries123@gmail.com"
                        
                        :: Force the remote origin URL to cleanly incorporate authentication variables
                        git remote set-url origin https://%GIT_USER%:%GIT_PASS%@://github.com
                        
                        :: =======================================================================
                        :: FUTURE REFERENCE: HOW TO SYNC ADDITIONAL FILES
                        :: =======================================================================
                        :: 1. Specific files: Append them to both lines separated by a space:
                        ::    git diff --quiet Jenkinsfile script.ps1 config.json
                        ::    git add Jenkinsfile script.ps1 config.json
                        ::
                        :: 2. Entire Workspace: Remove names completely to track everything:
                        ::    git diff --quiet
                        ::    git add .
                        :: =======================================================================
                        
                        :: Check if the local Jenkinsfile differs from the repository tracking index
                        git diff --quiet Jenkinsfile
                        if errorlevel 1 (
                            echo Local changes detected! Syncing upstream to GitHub...
                            git add Jenkinsfile
                            git commit -m "Automated Jenkinsfile sync from Jenkins Build #%BUILD_NUMBER% [skip ci]"
                            git push origin HEAD:main
                        ) else (
                            echo No local changes detected. Workspace is already perfectly in sync.
                        )
                    """
                }
            }
        }
    }
}
