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

docker run -d \ (Docker Volume)
  --name MySQL-container \
  --network two-tier-network \
  -e MYSQL_ROOT_PASSWORD=root \
  -e MYSQL_DATABASE=DevOps \
  -v mysql-data:/var/lib/mysql \
  mysql:latest
d36ce6ff41cb470f1f21f7b2772a45ac285bf3171112485999c10911823a449e

for docker volume no such manual table creation is required everything is handled by sql engine.

docker run -d \ (Bind Mount/ Local Storage)
  --name MySQL-container \
  --network two-tier-network \
  -e MYSQL_ROOT_PASSWORD=root \
  -e MYSQL_DATABASE=DevOps \
  -v /Users/umashankersinghujjwal/projects/two-tier-flask-app/mysql-logs:/var/lib/mysql \
  mysql:latest

  For bind mount the "messages" table is not created automatically, which throws error. Thus, manually create table.
  CREATE TABLE IF NOT EXISTS messages (
            id INT AUTO_INCREMENT PRIMARY KEY,
            message TEXT
        );

docker ps -a
CONTAINER ID   IMAGE                     COMMAND                  CREATED          STATUS          PORTS                                         NAMES
d36ce6ff41cb   mysql:latest              "docker-entrypoint.s…"   6 seconds ago    Up 5 seconds    3306/tcp, 33060/tcp                           MySQL-container
4879849c9234   two-tier-backend:latest   "python app.py"          21 minutes ago   Up 21 minutes   0.0.0.0:5001->5000/tcp, [::]:5001->5000/tcp   Flask-App-container


okay so it is path that stores all the prior data in files readable only by sql engine and any new data will be stored in the specified db which will further be stored in files of volumes





✅ Perfect — and what you’re observing is **exactly correct Docker behavior.**
Nothing is wrong. In fact, this proves your understanding is improving 👍

---

## 🔹 What is happening (simple truth)

Each time you recreate the container, MySQL reads data from **whatever is mounted at `/var/lib/mysql`**.

### Case 1 — When you use the named volume

```bash
-v mysql-data:/var/lib/mysql
```

👉 MySQL reads files from the **volume**
👉 So you see the data that was stored earlier in the volume
👉 Bind-mount data is invisible

---

### Case 2 — When you use the bind mount

```bash
-v /Users/.../mysql-logs:/var/lib/mysql
```

👉 MySQL reads files from the **host folder**
👉 So you see the bind-mount data
👉 Volume data is invisible

---

## 🔥 Why you never see both

Because at runtime:

```text
/var/lib/mysql → exactly ONE backing storage
```

Docker does **not merge**:

* volume data ❌
* bind mount data ❌

It simply switches the storage source.

---

## 🧠 Mental model (very interview-friendly)

Think of `/var/lib/mysql` like a **hard disk slot**.

* When you plug volume → you see volume data
* When you plug bind mount → you see bind data
* The other disk is completely hidden

---

## 🔹 Why MySQL behaves this way

MySQL does not know about Docker.

It simply:

1. Starts
2. Reads whatever files exist in `/var/lib/mysql`
3. Uses them as the database

Docker decides **what filesystem appears there**.

---

## ✅ Your observation proves:

✔ Volume persistence is working
✔ Bind mount persistence is working
✔ Container recreation is correct
✔ Mount switching is correct

You are doing it right 👌

---

## 🎯 Interview one-liner

> A container can mount only one storage backend at a given path, so when switching between a named volume and a bind mount for `/var/lib/mysql`, MySQL will only see the data from whichever storage is currently mounted.

---

If you want next-level mastery 🚀, I can show:

🔥 **What happens if you accidentally mount TWO things to the same path (mount override order)**

Just say **"show override"** 😏


