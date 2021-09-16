# Jenkins

## Install
brew install jenkins-lts

## Enable docker in Jenkins
Edit /usr/local/Cellar/jenkins-lts/2.303.1/homebrew.mxcl.jenkins-lts.plist

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>Label</key>
	<string>homebrew.mxcl.jenkins-lts</string>
	<key>ProgramArguments</key>
	<array>
		<string>/usr/local/opt/openjdk@11/bin/java</string>
		<string>-Dmail.smtp.starttls.enable=true</string>
		<string>-jar</string>
		<string>/usr/local/opt/jenkins-lts/libexec/jenkins.war</string>
		<string>--httpListenAddress=127.0.0.1</string>
		<string>--httpPort=8080</string>
	</array>
	<key>RunAtLoad</key>
	<true/>
	 <key>EnvironmentVariables</key>
    <dict>
      <key>PATH</key>
      <string>/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/Applications/Docker.app/Contents/Resources/bin/:/Users/Kh0a/Library/Group\ Containers/group.com.docker/Applications/Docker.app/Contents/Resources/bin</string>
    </dict>
</dict>
</plist>
```

## Start Jenkins
brew services start jenkins-lts
Open browser localhost:8080

## Check the initial password
cat /var/lib/jenkins/secrets/initialAdminPassword

## Create a project with Jenkinsfile
Example Jenkinsfile
```
pipeline {
    agent { docker { image 'node:14-alpine' } }
    stages {
        stage('build') {
            steps {
                sh 'npm --version'
            }
        }
    }
}
```

## Create a new item in Jenkins dashboard
link the project from github
Click the project
Click the branch
Build Now
Check the Console output


