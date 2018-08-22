docker build --no-cache --build-arg http_proxy=http://vipproxyprod.ingdirect.idi.it:9090 --build-arg https_proxy=http://vipproxyprod.ingdirect.idi.it:9090 -t braghettos/jenkins-slave-dind-jnlp .

docker run --name jenkins_slave -d --restart always --network=cdaas -v /var/run/docker.sock:/var/run/docker.sock --privileged braghettos/jenkins-slave-dind-jnlp -url http://jenkins_master.cdaas:8080 b132c536e530b137ddad60aa45f1e66b7cc7b395becc805faa563aab076a2c48 dind-node-1
