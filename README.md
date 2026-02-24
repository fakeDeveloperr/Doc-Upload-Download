Backend cannot connect to MySQL is why your container exits.

This happens because your containers are not on the same Docker network.
Why this happens

By default:

Each docker run uses the default bridge

Containers cannot resolve each other by name

DNS works only on user-defined networks

So your backend tries:


docker run -d --name MySQL-container --network two-tier-network -e MYSQL_ROOT_PASSWORD=root -e MYSQL_DATABASE=DevOps mysql:latest

docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS          PORTS                 NAMES
42de067536b0   mysql:latest   "docker-entrypoint.s…"   55 seconds ago   Up 54 seconds   3306/tcp, 33060/tcp   MySQL-container

docker run -d -p 5001:5000 --network two-tier-network -e MYSQL_HOST=MySQL-container -e MYSQL_USER=root -e MYSQL_PASSWORD=root -e MYSQL_DB=DevOps two-tier-backend:latest

docker ps
CONTAINER ID   IMAGE                     COMMAND                  CREATED         STATUS         PORTS                                         NAMES
77e4ede612b6   two-tier-backend:latest   "python app.py"          8 seconds ago   Up 7 seconds   0.0.0.0:5001->5000/tcp, [::]:5001->5000/tcp   naughty_sutherland
42de067536b0   mysql:latest              "docker-entrypoint.s…"   3 minutes ago   Up 3 minutes   3306/tcp, 33060/tcp                           MySQL-container



docker logs 77e4ede612b6
 * Serving Flask app 'app' (lazy loading)
 * Environment: production
   WARNING: This is a development server. Do not use it in a production deployment.
   Use a production WSGI server instead.
 * Debug mode: on
WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
 * Running on all addresses (0.0.0.0)
 * Running on http://127.0.0.1:5000
 * Running on http://172.19.0.3:5000


 docker inspect two-tier-network


Open in your Mac browser: This is where app is running
http://localhost:5001


docker exec -it MySQL-container bash
bash-5.1# mysql -u root -p 
Enter password: root
you can now access mysql

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| DevOps             |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.024 sec)

mysql> use devops;
ERROR 1049 (42000): Unknown database 'devops'

mysql> use DevOps;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> select * from messages; (messages is the table name where the input text from flask app is stored)
+----+---------+
| id | message |
+----+---------+
|  1 | hellooo |
|  2 | this    |
+----+---------+
2 rows in set (0.001 sec)

docker run -d \
  --name MySQL-container \
  --network two-tier-network \
  -e MYSQL_ROOT_PASSWORD=root \
  -e MYSQL_DATABASE=DevOps \
  -v mysql-data:/var/lib/mysql \
  -v /Users/umashankersinghujjwal/projects/two-tier-flask-app/mysql-logs:/var/log/mysql \
  mysql:latest
d36ce6ff41cb470f1f21f7b2772a45ac285bf3171112485999c10911823a449e

docker ps -a
CONTAINER ID   IMAGE                     COMMAND                  CREATED          STATUS          PORTS                                         NAMES
d36ce6ff41cb   mysql:latest              "docker-entrypoint.s…"   6 seconds ago    Up 5 seconds    3306/tcp, 33060/tcp                           MySQL-container
4879849c9234   two-tier-backend:latest   "python app.py"          21 minutes ago   Up 21 minutes   0.0.0.0:5001->5000/tcp, [::]:5001->5000/tcp   Flask-App-container


