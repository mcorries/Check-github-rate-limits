pipeline {
    agent any
    
    // PREVENTS JENKINS FROM AUTOMATICALLY OVERWRITING YOUR LOCAL WORKSPACE AT STARTUP
    options {
        skipDefaultCheckout()
    }
    
    environment {
        // Required for checkout scm (Username/Password format) and pushing back to GitHub
        GITHUB_CREDS = credentials('my-github-creds')
        // Fresh secret text token bound for your flexible API call
        GITHUB_TOKEN = credentials('Jenkins-Github')
    }          
    
    stages {
        stage('Automated Bidirectional Sync') {
            steps {
                echo "Checking for local workspace updates BEFORE checking out code..."
                
                // Using your credentials block to map user and password strings safely
                withCredentials([usernamePassword(credentialsId: 'my-github-creds', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_PASS')]) {
                    bat """
                        @echo off
                        :: Configure real production git signature for clean traceability
                        :: git config user.name "mcorries"
                        ::  git config user.email "mcorries123@gmail.com"
                        :: Change email to my official GitHub anonymous noreply ID address. Because GitHub controls this specific address format, it automatically cross-references it with your Jenkins-Github personal access token string during the push, applies the cryptographic signature on its backend servers, and turns the badge green automatically
                           git config user.name "mcorries"
                           git config user.email "mcorries@users.noreply.github.com"
                        
                        :: Force the remote origin URL to cleanly incorporate authentication variables without typos
                        git remote set-url origin https://%GIT_USER%:%GIT_PASS%@github.com/mcorries/Check-github-rate-limits.git
                        
                        :: =======================================================================
                        :: FUTURE REFERENCE: HOW TO SYNC ADDITIONAL FILES
                        :: =======================================================================
                        :: 1. Specific files: Append them to both lines separated by a space:
                        ::    git diff --quiet Jenkinsfile script.ps1 config.json README.md
                        ::    git add Jenkinsfile script.ps1 config.json README.md
                        ::
                        :: 2. Entire Workspace: Remove names completely to track everything:
                        ::    git diff --quiet
                        ::    git add .
                        :: =======================================================================
                        
                        :: NATIVE WINDOWS STATUS CHECK: Catches any status change (Staged, Unstaged, Untracked)
                        git status --porcelain Jenkinsfile README.md | findstr . >nul || type nul
                        if %errorlevel% equ 0 (
                            echo Local changes detected! Merging and syncing upstream to GitHub...
                            git add Jenkinsfile README.md
                            git commit -m "Automated workspace sync from Jenkins Build #%BUILD_NUMBER% [skip ci]"
                            
                            :: Pull and rebase down from the active upstream branch (master) to safely handle dual-sided updates
                            git pull --rebase origin master
                            
                            :: FIXED: Uses a fully qualified destination refname to prevent detached HEAD destination rejections
                            git push origin HEAD:refs/heads/master
                        ) else (
                            echo No local changes detected. Workspace is safe to refresh.
                        )
                    """
                }
            }
        }

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
                // PowerShell parsing block using the flexible %GITHUB_TOKEN% to do a raw query on the github API 
                bat "C:\\Windows\\System32\\curl.exe -s -H \"Accept: application/vnd.github.v3+json\" -H \"User-Agent: Jenkins-Pipeline\" -H \"Authorization: token %GITHUB_TOKEN%\" \"https://api.\"github\".com/rate_limit\" | powershell -Command \"\$input | ConvertFrom-Json | ForEach-Object { \$_.resources.PSObject.Properties | ForEach-Object { \$time = [System.DateTimeOffset]::FromUnixTimeSeconds(\$_.Value.reset).LocalDateTime.ToString('yyyy-MM-dd HH:mm:ss'); Write-Output ('Stage: ' + \$_.Name.ToUpper().PadRight(28) + ' | Remaining: ' + \$_.Value.remaining.ToString().PadRight(5) + ' / ' + \$_.Value.limit.ToString().PadRight(6) + ' | Resets At: ' + \$time) } }\""
            }
        }
    }
}