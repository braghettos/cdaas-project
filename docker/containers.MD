docker run --name artifactory -d -p 8082:8081 --memory=1g --restart always -e EXTRA_JAVA_OPTIONS=-Xmx1g --network=cdaas docker.bintray.io/jfrog/artifactory-pro:latest


docker run --name gitlab -d --hostname gitlab.cdaas -p 8083:80  --memory=2g --restart always --env GITLAB_OMNIBUS_CONFIG="external_url 'http://gitlab.cdaas/'; unicorn['worker_processes'] = 2; sidekiq['concurrency'] = 1; prometheus_monitoring['enable'] = false; gitlab_rails['env'] = {'http_proxy' => \"http://vipproxyprod.ingdirect.idi.it:9090\", 'https_proxy' => \"http://vipproxyprod.ingdirect.idi.it:9090\", 'no_proxy' => \"cdaas\", 'HTTP_PROXY' => \"http://vipproxyprod.ingdirect.idi.it:9090\", 'HTTPS_PROXY' => \"http://vipproxyprod.ingdirect.idi.it:9090\", 'NO_PROXY' => \"cdaas\"}; omnibus_gitconfig['system'] = { 'http' => [\"sslVerify = false\"] }; " -e http_proxy=http://vipproxyprod.ingdirect.idi.it:9090 -e https_proxy=http://vipproxyprod.ingdirect.idi.it:9090 -e no_proxy=cdaas --network=cdaas --volume gitlab-config:/etc/gitlab --volume gitlab-logs:/var/log/gitlab --volume gitlab-data:/var/opt/gitlab gitlab/gitlab-ce:11.1.4-ce.0


docker run --name postgres -d --memory=512m --restart always -e SECRET_KEY="awxsecret" -e POSTGRES_DB="awx" -e POSTGRES_USER="awx" -e POSTGRES_PASSWORD="awxpass" --network=cdaas postgres:9.6


docker run --name rabbitmq -d --memory=512m --restart always -e RABBITMQ_DEFAULT_VHOST="awx" --network=cdaas rabbitmq:3


docker run --name memcached -d --memory=128m --restart always --network=cdaas memcached:alpine


docker run --hostname awx --name awx_task -d --memory=1g --restart always -e SECRET_KEY="awxsecret" -e DATABASE_NAME="awx" -e DATABASE_USER="awx" -e DATABASE_PASSWORD="awxpass" -e DATABASE_PORT="5432" -e DATABASE_HOST="postgres" -e RABBITMQ_USER="guest" -e RABBITMQ_PASSWORD="guest" -e RABBITMQ_HOST="rabbitmq" -e RABBITMQ_PORT="5672" -e RABBITMQ_VHOST="awx" -e MEMCACHED_HOST="memcached" -e MEMCACHED_PORT="11211" --network=cdaas ansible/awx_task:1.0.6


docker run --hostname awxweb --name awx_web -d --memory=1g --restart always -e SECRET_KEY="awxsecret" -e DATABASE_NAME="awx" -e DATABASE_USER="awx" -e DATABASE_PASSWORD="awxpass" -e DATABASE_PORT="5432" -e DATABASE_HOST="postgres" -e RABBITMQ_USER="guest" -e RABBITMQ_PASSWORD="guest" -e RABBITMQ_HOST="rabbitmq" -e RABBITMQ_PORT="5672" -e RABBITMQ_VHOST="awx" -e MEMCACHED_HOST="memcached" -e MEMCACHED_PORT="11211" --network=cdaas -p 8052:8052 -e HTTP_PROXY=http://vipproxyprod.ingdirect.idi.it:9090 -e HTTPS_PROXY=http://vipproxyprod.ingdirect.idi.it:9090 ansible/awx_web:1.0.6


docker run -d -v jenkins_home:/var/jenkins_home -p 8084:8080 -p 50000:50000 --restart=always --memory=2g --name=jenkins_master -e HTTP_PROXY=http://vipproxyprod.ingdirect.idi.it:9090 -e HTTPS_PROXY=http://vipproxyprod.ingdirect.idi.it:9090 --env JAVA_OPTS="-Dhttp.proxyHost=vipproxyprod.ingdirect.idi.it -Dhttp.proxyPort=9090" --network=cdaas --user=root  jenkins/jenkins:lts


docker run --name jenkins_slave -d --restart always --network=cdaas --user=root -v /var/run/docker.sock:/var/run/docker.sock --privileged jenkins/jnlp-slave -url http://jenkins_master:8080 35c3f75cdbfbf20c59ec2a2aa24663fc2eb940c0b9803c7d97cd73d30c4aac28 docker-slave


docker run --name jenkins_slave -d --memory=512m --restart always --network cdaas -v /var/run/docker.sock:/var/run/docker.sock -v maven-repo:/root/.m2/ -v jenkins_slave_home:/var/jenkins_home --privileged --user root braghettos/jenkins-slave-dind-jnlp:1.12.6 -url http://jenkins_master.cdaas:8080 b132c536e530b137ddad60aa45f1e66b7cc7b395becc805faa563aab076a2c48 dind-node-1


