# Containered Magento Dev Environment

The following setup will create a dev environment on your local computer, saving you to run any of the services you usually need to operate a working environment.

This dev environment is using the following programs as containers, and therefore **prior to starting this setup, you will need to stop any of those services** running on your machine or alternatively change their default ports, otherwise the containers won't be able to run successfully, as all services are using the default ports:  
* PHP 7.4  with Apache 2.4 (and Node 14 installed)  
* MySQL 8.0.28  
* Redis 6  
* Elasticsearch 7.1  
* RabbitMQ 3.8  
* Kibana 7.11  
* phpMyAdmin 5.1  
* MailHog  

After performing the following setup steps, you will be able to perform your regular development and debugging tasks as usual. All data modifications both on the file system and the database will be permanent and won't be lost between starts and stops of this environment.

Also **Git** will work as usual, both from your CLI as well as from your IDE.

> Note! <br>
> This setup is built for development only, and not for production use, as all the ports including the php ports are open to 0.0.0.0! Therefore **don't deploy it on a VM accessible from the internet**. (For those purposes, you may run the `docker-compose-prod.yml` which only exposes ports 80 and 443 for apache and 8080 for phpMyAdmin, and all other ports will be only accessible via a local subnet exposed in the docker backend network defined.)
## Setup

### Docker Install

Make sure that Docker is installed on your computer. On Mac the application running will be Docker Desktop.

Ensure in the settings page (of the Docker Desktop) that the memory limit for docker processes is set to the maximum (and not the default 2 GB).

### Docker-Magento Repo

* Clone this repo to the desired directory on your local computer.
* Change the name of the repo folder to the name of your project.

### Project Repo

* Enter the root path of the Docker-Magento repo downloaded (whose name you just changed).
* Change directory into the www folder. 
* Delete the file `.gitkeep`.
* Clone your **Magento project repo** into this folder (putting a '.' at the end.)
* Download your **env.php** and **config.php** files into **www/app/etc**. 
* Open your env.php file and set the following values for key db:
* * host: `db`
* * dbname: `magento`
* * username: `root`
* * password: `anat2509`
* In addition, search the file for appearances of `redis`, and if found, change the host to `redis` and it's port to `6379` (in case it's different)

### Run your containers

* From the root path of the Docker-Magento project (i.e. above www), execute in the terminal `bin/start`.  
For your first dev environment this process might take a few minutes, and it will be completed, when the containers will be all up and you will be returned to the command prompt.

### Import the project database

* Enter phpMyAdmin at http://localhost:8080 and verify that a database named `magento` already exists.
* Download your DB dump to the Docker-Magento project folder (i.e. the folder above www) and run the following command (specifying your actual database name):

`bin/mysql < database/dump-file-name.sql`

The duration of the import might take some time and will be finished when you will be returned to the command prompt.

After completion, your imported database should show up within database `magento`.

Edit your local hosts file and add a host name for your server like '127.0.0.1 project.loc' (i.e. maccabi.loc, or unilever.loc, or bezeq.loc ...)

Enter table `core_config_data` and change the following values according to the specified path:
* `web/unsecure/base_url` and `web/secure/base_url` (as long as it exists): `http://project.loc` (i.e. your hostname set in your hosts file). 
* `catalog/search/elasticsearch7_server_hostname`: `elasticsearch`
* `catalog/search/elasticsearch7_server_port`: `9200`  
In addition, search in path for `elasticsearch` and verify that ElasticSearch7 is set, and not version 6. If you find  projects like **Unilever-ff** which point still to ElasticSearch 6, you will need to change the version in the path from 6 to 7. 
Also verify that the path `catalog/search/engine` is set to `elasticsearch7`. (Again, this seems to be only relevant for our Unilever project.)

### Composer Install and Magento Setup

Within the Docker-Magento project root directory enter the following commands:  
* `bin/composer install` (This will take quite a time.)  
* `bin/magento setup:upgrade`  
* `chmod -R 777 www`  
* `bin/magento setup:di:compile`  

In your browser enter "http://project.loc/admin" (or /xmng instead of /admin (i.e. in the case of Bezeq)) and you should (after some time) enter successfully your back office.

## Workflow

### Starting and Stopping your Dev Environment / Trouble Shooting

You easily can stop your environment by running `bin/stop` and you can start it by running `bin/start`.

You can *check the status* of the environment by running `bin/status` and you will receive a list of 7 containers, and as long as those containers are in status UP, everything should run ok.

If the state of one or all containers is not UP, you can run `bin restart`.

### Database Credentials

* DB Host: `db`
* DB Name: `magento`
* DB Username: `root`
* DB Password: `anat2509`
> Note! <br>
> All you database data and file system changes will be saved!

### Mailhog as SMTP server

In this development environment, sending mail via the php mail() function will be automatically routed to Mailhog, a SMTP server for testing purposes, whose UI is running at port 8025 of your server (i.e. http://localhost:8025). When accessing it you will be asked for your username, which is magento, and your password, which is anat2509. The list of sent mails from your dev environment will be shown.

### Setting up additional Dev Environments

Can be done easily by performing the described setup steps, and you only need to choose a diffent project path, and to stop your current dev environment before starting any other dev environments.

Therefore also *switching between Dev Environments* is straight forward, and you just need to run the `bin/start` and `bin/stop` commands from your project root directory (i.e. where the file `docker-compose.yml` is located), or specify the correct path to your project folder with those two commands.

*Only one Dev Environment* can run at any given time (or at least as long as the ports of the services are the default ones).

### Useful commands

`bin/status` will show you 7 container statuses.  
`bin/start` will start your dev environment.  
`bin/stop` will stop all containers of your dev environment.  
`bin/magento` will run your magento commands.  
`bin/cli` will enter you into the main container (running php-fpm and node) within your main project directory.  
`bin/composer` will run composer commands.  
`bin/mysql` will give you the mysql command line (using user root).  
`bin/mysqldump > db.sql` will dump your magento database into file db.sql (or any file name you chose) on your project root folder.  
`bin/cache-clean` will flush all magento caches. You may also add cache types like config full_page.  
`bin/redis`: Run a command from the redis container. Ex. `bin/redis redis-cli monitor`.  
`bin/node` will give you the Node REPL.  
`bin/grunt` will execute grunt from your project root folder (after setting up Grunt).  

### Troubleshooting

In case, something stops functioning as usual, run `bin/status` and if one of the containers is not in state UP, you will see the Exit Code listed, and only an exit code of 0 would indicate that you intentionally stopped the specific container, but an exit code of 137 would indicate that docker ran out of memory, and as a consequence terminated a container taking too many resources (like ElasticSearch). 

In such a case, `bin/start` will start the container stopped, but eventually you will solve a memory problem only by enlarging the physical memory of the machine.

If you want to see the resource usage of the individual containers, running `docker stats` will show it to you.
