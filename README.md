# Demo Django Application with MYSQL and Nginx Reverse Proxy.
 ![Dockerize-01](https://github.com/jodeh/Dockerized-Django-Nginx-MYSQL/assets/80529706/028acd0c-95be-4a68-8b1a-9550db43c9ef)
<hr>

## Table of Contents
 * ### Prerequisites
 * ### How to build
 * ### Configuration
<hr>

### Prerequisites
Make sure you have downloaded these prerequisites before we start 
* [Docker](https://docs.docker.com/get-docker/)
* [Docker compose](https://docs.docker.com/compose/install/)
<hr>

### How to build
 
  1. Clone the repository to your local machine
     ```
     git clone https://github.com/jodeh/Dockerized-Django-Nginx-MYSQL.git
     cd Dockerized-Django-Nginx-MYSQL
     ```
  2. Before we start to run the containers, we need to make a network to let the containers to connect with each other
     ```
     docker network create --subnet 10.0.0.0/24 custom-network
     ```
  3. Now lets run our mysql container :
     ```
     docker run -d --network custom-network --ip 10.0.0.15 --hostname mysql-database -e MYSQL_ROOT_PASSWORD=password -e MYSQL_DATABASE=djangodb mysql
     ```
  4. Build the Django image and run the container
     ```
     cd /Django
     docker build -t django-app .
     docker run -d --network custom-network --ip 10.0.0.10 --hostname django django-app
     ```
  5. Build the nginx image and run it
       ```
       cd /nginx
       docker build -t nginx-reverse-proxy .
       docker run -d --network custom-network --ip 10.0.0.5 -p 80:80 --hostname nginx nginx-reverse-proxy  
        ```
     
  6. Create user in mysql .
       ```
       docker exec -it <mysql container id> bash
       
       $ mysql -p
       > CREATE USER 'jodeh'@'%' IDENTIFIED BY 'password';
       > GRANT ALL PRIVILEGES ON *.* TO 'jodeh'@'%' WITH GRANT OPTION;
       > FLUSH PRIVILEGES;
       ```
 7. Migrate tables to the new database
       ```
       docker exec -it <django container id> bash
       cd ~/mysite
       python3 manage.py migrate
       ```
  7. Now go back to mysql container to check tables
     ```
     docker exec -it <mysql container id> bash
     
     $ mysql -p
     > SHOW DATABASES;
     > USE djangodb;
     > SHOW TABLES;
     ```
     Tables must be shown like this.
   
     ![djangodb](https://github.com/jodeh/Dockerized-Django-Nginx-MYSQL/assets/80529706/e7de7bf2-4c24-4a4a-858a-48264800cbfd)

     
   9. Finally Check The Connectivity
      
   ![Django](https://github.com/jodeh/Dockerized-Django-Gunicorn-Postgres-Nginx/assets/80529706/cf90de2b-9d61-439a-b077-6a843e377d29)
   
<hr>

### Configuration
  * We can configure django settings by editting ./django/mysite/settings.py file
  * The proxy pass and the server name can be configured by editting ./etc/nginx/conf.d/default.conf file
