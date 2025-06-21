# Register App Deployment
## CI Pipeline
### Jenkins Plugins
- Maven Integration
- Pipeline Maven Integration
- Eclipse Temurin Installer
![Jenkins Maven Plugins](./imgs/jenkins-maven-plugins.png)
### Jenkins Tools
![Maven Tool Setup](./imgs/setup-maven-tool.png)
![JDK Tool Setup](./imgs/setup-jdk-tool.png)


## Trivy Setup
```
sudo apt-get install wget gnupg
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | gpg --dearmor | sudo tee /usr/share/keyrings/trivy.gpg > /dev/null
echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb generic main" | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy
```

## SonarQube Setup
```
sudo docker run -d --name sonarqube -p 9000:9000 sonarqube:lts
```