pipeline {
    agent any
    
    environment {
        GITHUB_CREDS = credentials('my-github-creds')
    }          
    
    stages {
        stage('Safe Source Checkout') {
            steps {
                echo "Starting safe repository checkout..."
                // Using the built-in scm object ensures it pulls the exact commit that triggered the webhook
                checkout scm
                echo "Checkout complete."
            }
        }

        stage('Verify GitHub Auth & Rate Limit') {
            steps {
                echo "Checking GitHub authentication for user: ${env.GITHUB_CREDS_USR}"
                echo "------------------------------------------------------------------------"
                
                // Native curl passes headers safely. Quote-chopping \"github\" stops Jenkins URL rewriting.
                bat "C:\\Windows\\System32\\curl.exe -s -H \"Accept: application/vnd.github.v3+json\" -H \"User-Agent: Jenkins-Pipeline\" -H \"Authorization: token %GITHUB_CREDS_PSW%\" \"https://api.\"github\".com/rate_limit\" | powershell -Command \"\$input | ConvertFrom-Json | ForEach-Object { \$_.resources.PSObject.Properties | ForEach-Object { \$time = [System.DateTimeOffset]::FromUnixTimeSeconds(\$_.Value.reset).LocalDateTime.ToString('yyyy-MM-dd HH:mm:ss'); Write-Output ('Stage: ' + \$_.Name.ToUpper().PadRight(28) + ' | Remaining: ' + \$_.Value.remaining.ToString().PadRight(5) + ' / ' + \$_.Value.limit.ToString().PadRight(6) + ' | Resets At: ' + \$time) } }\""
                
                echo "------------------------------------------------------------------------"
                echo "REVERSE TEST: GitHub Web Interface Edit Successfully Triggered Jenkins!"
            }
        }
    }
}
