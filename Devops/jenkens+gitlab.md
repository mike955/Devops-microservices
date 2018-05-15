# jenkins+gitlab 持续集成

docker脚本如下
```sh
docker pull jenkins
docker itlab/gitlab-ce

docker run --name myjenkins -p 8090:8080 -p 50000:50000 -v /var/jenkins_home jenkins

docker run --detach \
    --hostname gitlab.example.com \
    --publish 443:443 --publish 800:80 --publish 220:22 \
    --name gitlab \
    --restart always \
    --volume /Users/clx/dockerVolume/gitlab/config:/etc/gitlab \
    --volume /Users/clx/dockerVolume/gitlab/logs:/var/log/gitlab \
    --volume /Users/clx/dockerVolume/gitlab/data:/var/opt/gitlab \
    gitlab/gitlab-ce:latest
```