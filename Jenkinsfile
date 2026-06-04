pipeline {
    agent any
    
    environment {
        // Required for checkout scm (Username/Password format)
        GITHUB_CREDS = credentials('my-github-creds')
        // Fresh secret text token bound for your flexible API call
        GITHUB_TOKEN = credentials('Jenkins_Github')
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

               echo "------------------------------------------------------------------------"
               echo "REVERSE TEST: GitHub Web Interface Edit Successfully Triggered Jenkins!" 
            }
        }
    }
}
