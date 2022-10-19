**Homepage:**[Jenkins.io](https://www.jenkins.io/)
**Documentation**: [User Guide](https://www.jenkins.io/doc/book/installing/)

## Steps

**Install OpenJDK 11**

```bash
sudo apt update
sudo apt install openjdk-11-jre
java -version
openjdk version "11.0.12" 2021-07-20
OpenJDK Runtime Environment (build 11.0.12+7-post-Debian-2)
OpenJDK 64-Bit Server VM (build 11.0.12+7-post-Debian-2, mixed mode, sharing)

```

**Install Docker**

```bash
curl -fsSL get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo systemctl enable docker
sudo usermod -aG docker <User>
docker run hello-world
```

**Install Jenkins Service LTS Version**

A [LTS (Long-Term Support) release](https://www.jenkins.io/download/lts/) is chosen every 12 weeks from the stream of regular releases as the stable release for that time period. It can be installed from the [`debian-stable` apt repository](https://pkg.jenkins.io/debian-stable/).

```bash
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
sudo systemctl status jenkins
```

To get the initial password, just exceute
```bash
cat /var/lib/jenkins/secrets/initialAdminPassword
```

**Add** [Docker Pipeline Plugin](https://plugins.jenkins.io/docker-workflow/)

**Example Jenkins Pipeline Script**

```groovy
pipeline {
    agent {
        docker { image 'node:16.13.1-alpine' }
    }
    stages {
        stage('Test') {
            steps {
                sh 'node --version'
            }
        }
    }
}
```

[More examples](https://www.jenkins.io/doc/book/pipeline/docker/)