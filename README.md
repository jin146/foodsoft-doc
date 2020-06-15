# foodsoft-doc
## Docker

- Install docker and launch docker daemon

			curl -sSL https://get.docker.com/ | sh
		
			sudo dockerd

- setup mysql container and get it's IP

			docker pull mysql:5.5

			docker run --name mysql -e MYSQL_ROOT_PASSWORD=mysql -d mysql:5.5

			docker exec -it mysql mysql -u root -p
			
	It is recommended to run the mysql container in a way that preserves your database in case the container is shotdown
			
			mkdir /home/user/mysql
			chmod 775 /home/user/mysql
	(not sure for the rights)
	
  			docker run --name mysql -e MYSQL_ROOT_PASSWORD=mysql -d --volume=/home/user/mysql:/var/lib/mysql mysql:5.5
			
			CREATE database foodsoftdb;
            		CREATE user 'foodsoft';			
            		GRANT ALL ON `foodsoftdb`.* TO foodsoft;
			FLUSH PRIVILEGES;

			docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' mysql

- setup redis container and get it's IP

			docker pull redis:3.2

			docker run --name redis redis:3.2

			docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' redis

- setup foodsoft container

    - first locally edit app_config.yml and send it to the server

			scp app_config.yml root@host.net:/home/user

    - then launch foodsoft project's containers

			docker pull foodcoops/foodsoft:4.6.0

			docker run --name foodsoft_web -p 3000 -e SECRET_KEY_BASE='mYunBreAkaBleKEY15476147654145' -e DATABASE_URL='mysql2://root:mysql@172.17.0.2/foodsoftdb?encoding=utf8' -e REDIS_URL='redis://172.17.0.4:6379' -e RAILS_FORCE_SSL=false   -v /home/user/app_config.yml:/usr/src/app/config/app_config.yml:ro   foodcoops/foodsoft:4.6.0

			docker run --name foodsoft_setup --rm  -e SECRET_KEY_BASE='mYunBreAkaBleKEY15476147654145' -e DATABASE_URL='mysql2://root:mysql@172.17.0.2/foodsoftdb?encoding=utf8' -e REDIS_URL='redis://172.20.0.4:6379' -v /home/user/app_config.yml:/usr/src/app/config/app_config.yml:ro   foodcoops/foodsoft:4.6.0 bundle exec rake db:setup

            docker run --name foodsoft_worker  -e SECRET_KEY_BASE='mYunBreAkaBleKEY15476147654145' -e DATABASE_URL='mysql2://root:mysql@172.17.0.2/foodsoftdb?encoding=utf8' -e REDIS_URL='redis://172.17.0.4:6379' -v /home/user/app_config.yml:/usr/src/app/config/app_config.yml:ro   foodcoops/foodsoft:4.6.0 ./proc-start worker
	    
			docker run --name foodsoft_smtp -e SECRET_KEY_BASE='mYunBreAkaBleKEY15476147654145' -e DATABASE_URL='mysql2://root:mysql@172.17.0.2/foodsoftdb?encoding=utf8' -e REDIS_URL='redis://172.17.0.4:6379' -v /home/user/app_config.yml:/usr/src/app/config/app_config.yml:ro   foodcoops/foodsoft:4.6.0 ./proc-start mail
	    
			docker run --name foodsoft_cron -e SECRET_KEY_BASE='mYunBreAkaBleKEY15476147654145' -e DATABASE_URL='mysql2://root:mysql@172.17.0.2/foodsoftdb?encoding=utf8' -e REDIS_URL='redis://172.17.0.4:6379' -v /home/user/app_config.yml:/usr/src/app/config/app_config.yml:ro   foodcoops/foodsoft:4.6.0 ./proc-start cron

### Useful docker commands

- Stop and remove all containers

            docker stop $(docker ps -a -q)

            docker rm $(docker ps -a -q)

- remove all downloaded images

            docker rmi $(docker images -q)
            docker rmi $(docker images -q) --force

## Docker-compose draft

            sudo curl -L "https://github.com/docker/compose/releases/download/1.23.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
