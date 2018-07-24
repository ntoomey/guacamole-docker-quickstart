sudo apt update

sudo apt install docker.io -y

sudo docker network create --driver bridge guac-network

sudo systemctl restart docker

sudo docker run --name guacd -d --network guac-network guacamole/guacd 

sudo docker run --name mysql -v /srv/mysql:/var/lib/mysql -d -e MYSQL_DATABASE=guacamole -e MYSQL_ALLOW_EMPTY_PASSWORD=yes --network guac-network mysql:5.7.22

sudo docker exec -i mysql mysql -u root --database mysql --execute="create user 'guacuser'@'%' identified by 'blank';"

sudo docker exec -i mysql mysql -u root --database mysql --execute="grant all privileges on guacamole.* to 'guacuser'@'%';"

sudo docker run --name guacamole -d -p 8080:8080 -e GUACD_HOSTNAME=guacd -e GUACD_PORT=4822 -e MYSQL_HOSTNAME=mysql -e MYSQL_USER=guacuser -e MYSQL_DATABASE=guacamole -e MYSQL_PASSWORD=blank --network guac-network guacamole/guacamole

sudo docker run --rm guacamole/guacamole /opt/guacamole/bin/initdb.sh --mysql > initdb.sql

sudo docker exec -i mysql mysql -u root guacamole < initdb.sql

