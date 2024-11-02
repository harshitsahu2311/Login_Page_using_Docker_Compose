

![image](https://github.com/user-attachments/assets/3278cd2c-b241-4a7b-ac36-10d51662c59c)

## Projects Video
<table>
  <tr>
    <td>
      <a href="https://www.youtube.com/watch?v=VcZaCSAO1wo" target="_blank">
        <img src="https://ytcards.demolab.com/?id=VcZaCSAO1wo&title=Part+-+32.+Created+CI%2FCD+Pipeline+For+Flask+Application+%7C+DevOps+Project+%7C+%40SenDevOps&lang=en&timestamp=0&background_color=%230d1117&title_color=%23ffffff&stats_color=%23dedede&max_title_lines=1&width=250&border_radius=5" 
        alt="Part - 32. Created CI/CD Pipeline For Flask Application | DevOps Project | @SenDevOps" width="250" style="border-radius:5px;">
      </a>
    </td>
    <td>
      <a href="https://www.youtube.com/watch?v=jRUFai_M-C0" target="_blank">
        <img src="https://ytcards.demolab.com/?id=jRUFai_M-C0&title=Part+-+31.+Deploy+Flask+App+Using+Docker+Compose+%7C+DevOps+Project+%7C+%40SenDevOps&lang=en&timestamp=0&background_color=%230d1117&title_color=%23ffffff&stats_color=%23dedede&max_title_lines=1&width=250&border_radius=5" 
        alt="Part - 31. Deploy Flask App Using Docker Compose | DevOps Project | @SenDevOps" width="250" style="border-radius:5px;">
      </a>
    </td>
    <td>
      <a href="https://www.youtube.com/watch?v=-HfM4pe2NAw&t=61s" target="_blank">
        <img src="https://ytcards.demolab.com/?id=-HfM4pe2NAw&title=Part+-+30.+Deploy+Flask+App+With+Docker+%7C+DevOps+Projects+%7C+%40SenDevOps&lang=en&timestamp=0&background_color=%230d1117&title_color=%23ffffff&stats_color=%23dedede&max_title_lines=1&width=250&border_radius=5" 
        alt="Part - 30. Deploy Flask App With Docker | DevOps Projects | @SenDevOps" width="250" style="border-radius:5px;">
      </a>
    </td>
    <td>
      <a href="https://www.youtube.com/watch?v=IIC3LfnyOJI&t=47s" target="_blank">
        <img src="https://ytcards.demolab.com/?id=IIC3LfnyOJI&title=Part+-+29.+Deploy+Flask+App+on+EC2+Using+MySQL+Database+%7C+%40SenDevOps&lang=en&timestamp=0&background_color=%230d1117&title_color=%23ffffff&stats_color=%23dedede&max_title_lines=1&width=250&border_radius=5" 
        alt="Part - 29. Deploy Flask App on EC2 Using MySQL Database | @SenDevOps" width="250" style="border-radius:5px;">
      </a>
    </td>
  </tr>
</table>



# Part -1. Run in Local Server or EC2
## Install MySQL on the EC2 Instance
```
sudo apt update
sudo apt install mysql-server -y
```

## Configure MySQL
- Start the MySQL service
```
sudo systemctl start mysql
sudo systemctl enable mysql
```
- Set the MySQL root password:
```
sudo mysql
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';
FLUSH PRIVILEGES;
```
- Create the database, user and table:
```
CREATE DATABASE mydb;
USE mydb;

CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL,
    password VARCHAR(255) NOT NULL
);

CREATE USER 'root'@'%' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON mydb.* TO 'root'@'%';
FLUSH PRIVILEGES;

```
## Allow Remote Connections to MySQL
- Edit the MySQL configuration file:
```
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
```
- Change it to:
```
bind-address = 0.0.0.0
```
- Restart MySQL:
```
sudo systemctl restart mysql
```



## Run Your Flask App
- Install the required Python packages:
```
pip install flask mysql-connector-python werkzeug

```
- Start the Flask application:
```
python3 app.py
```
---

# Part-2. Dockerize
## Step 1: Create a Docker Network
- First, create a custom Docker network to allow communication between the MySQL and Flask containers.
```
docker network create my-network
```
## Step 2: Create and Run the MySQL Container
```
docker run -d \
  --name mysql_container \
  --network my-network \
  -e MYSQL_ROOT_PASSWORD=password \
  -e MYSQL_DATABASE=mydb \
  -p 3306:3306 \
  mysql:8.0
```
### Verify MySQL Setup
- Check if the MySQL container is running:
```
docker ps -a
```
- Enter the MySQL container to check if the database was created:
```
docker exec -it mysql_container mysql -u root -p
```
- Use `password` when prompted and run:
```
SHOW DATABASES;
USE mydb;
```
## Step 3: Update Flask App's `app.py`
```
db = mysql.connector.connect(
    host="mysql_container",  # MySQL container's name
    user="root",
    password="password",
    database="mydb"
)
```
- Create Table is not present
```
CREATE DATABASE mydb;

USE mydb;

CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL,
    password VARCHAR(255) NOT NULL
);

CREATE USER 'root'@'%' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON mydb.* TO 'root'@'%';
FLUSH PRIVILEGES;

```
## Step 4: Create Dockerfile for Flask App
```
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install -r requirements.txt

COPY . .

EXPOSE 5000

CMD ["python3","app.py"]
```
## Step 5: Build and Run the Flask App Container
- Build the Flask app image:
```
docker build -t flask-app .
```
- Run the Flask app container connected to the same Docker network:
```
docker run -d \
  --name flask_app_container \
  --network my-network \
  -p 5000:5000 \
  flask-app
```
## Step 6: Test the Application
```
http://<your-ec2-public-ip>:5000
```
# Run Using Docker-Compose File
- Install Docker Compose
```
apt install docker-compose
```
- Run the Application
```
docker-compose up -d
```
- Down the Container
```
docker-compose down 
```
