---
tags: ["jenkins"]
---

# jenkins本地部署

## 开启服务

```shell
docker run -d -u root -p 8081:8080 -v jenkins-data:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock --name jenkins jenkins/jenkins:lts
```

默认安装

## 额外安装docker插件

- Docker Pipeline
- Docker Commons

## 进入容器安装docker服务

`docker exec -it jenkins bash`

```shell
apt-get update
apt-get install -y docker.io
```

## 测试

```shell
pipeline {
    agent { docker 'python:3.5.1' }
    stages {
        stage('build') {
            steps {
                sh 'python --version'
            }
        }
    }
}
```
