# Drupal-nginx-php Docker
This is a Drupal Docker image which can run on both 
 - [Azure Web App on Linux](https://docs.microsoft.com/en-us/azure/app-service-web/app-service-linux-intro)
 - [Drupal on Linux Web App With MySQL](https://ms.portal.azure.com/#create/Drupal.Drupalonlinux )
 - Your Docker engines's host.

You can find it in Docker hub here [https://hub.docker.com/r/appsvcorg/drupal-nginx-fpm/](https://hub.docker.com/r/appsvcorg/drupal-nginx-fpm/)

# Components
This docker image currently contains the following components:
1. Drupal (Git pull as you wish)
2. nginx (1.14.0)
3. PHP (7.2.9)
4. Composer (1.7.2)
5. Drush (9.4.0)
6. MariaDB ( 10.1.26/if using Local Database )
7. Phpmyadmin ( 4.8.0/if using Local Database )

## How to Deploy to Azure 
1. Create a Web App for Containers, set Docker container as ```appsvcorg/drupal-nginx-fpm:php7.2.9``` 
   OR: Create a Drupal on Linux Web App With MySQL.
2. Add one App Setting ```WEBSITES_CONTAINER_START_TIME_LIMIT``` = 900
3. Browse your site and wait almost 10 mins, you will see install page of Drupal.
4. Complete Drupal install.

## How to configure GIT Repo and Branch
1. Create a Web App for Containers
2. Add new App Settings

Name | Default Value
---- | -------------
GIT_REPO | https://github.com/azureappserviceoss/drupalcms-azure
GIT_BRANCH | linuxappservice-composer

4. Browse your site

>Note: GIT directory: /home/site/wwwroot.
>
>Note: ```WEBSITES_ENABLE_APP_SERVICE_STORAGE``` = false, Before restart web app, need to store your changes by "git push", it will be pulled again after restart.
>
>Note: ```WEBSITES_ENABLE_APP_SERVICE_STORAGE``` = true, and /home/site/wwwroot/sites/default/settings.php is exist, it will not pull again after restart.


## How to configure to use Local Database with web app 
1. Create a Web App for Containers 
2. Update App Setting ```WEBSITES_ENABLE_APP_SERVICE_STORAGE``` = true
3. Add new App Settings 

Name | Default Value
---- | -------------
DATABASE_TYPE | local
DATABASE_USERNAME | some-string
DATABASE_PASSWORD | some-string

>Note: We create a database "azurelocaldb" when using local mysql. Hence use this name when setting up the app.

4. Browse http://[website]/phpmyadmin

# How to turn on Xdebug to profile the app
1. By default Xdebug is turned off as turning it on impacts performance.
2. Connect by SSH.
3. Go to ```/usr/local/etc/php/conf.d```,  Update ```xdebug.ini``` as wish, don't modify the path of below line.
```zend_extension=/usr/local/lib/php/extensions/no-debug-non-zts-20170718/xdebug.so```
4. Save ```xdebug.ini```, Restart php-fpm by below cmd:
```
# find gid of php-fpm
ps aux
# Kill master process of php-fpm
kill -INT <gid>
# start php-fpm again
php-fpm -D
chmod 777 /run/php/php7.0-fpm.sock
```
5. Xdebug is turned on.

# Enable redis support 
1. Deploy to Azure, Complete Drupal install.
2. Admin -> Extend, set the checkbox of Redis as true, click Install button.
3. Run below cmd: 
```
cd /home/drupal_prj/web/sites/default
chmod 777 settings.php
cat settings.redis.php >> settings.php
```
4. Refresh Browser, Admin -> Reports -> Status report, REDIS in Checked list, it's connected.

# Choose Listen Type of php-fpm/nginx
1. By default, ```LISTEN_TYPE``` = socket.
2. Update App Setting ```LISTEN_TYPE``` = port if you perfer to listening from TCP/IP.

# Updating Drupal version , themes , files 

If ```WEBSITES_ENABLE_APP_SERVICE_STORAGE``` = false  ( which is the default setting ), we recommend you DO NOT update the Drupal core version, themes or files.

There is a tradeoff between file server stability and file persistence. Choose either one option to updated your files:

##### OPTION 1 : 
Since we are using local storage for better stability for the web app , you will not get file persistence.  In this case , we recommend to follow these steps to update WordPress Core  or a theme or a Plugins version:
1.	Fork the repo https://github.com/leonzhang77/drupalcms-azure
2.	Clone your repo locally and make sure to use ONLY linuxappservice branch
3.	Download the latest version of Drupal , plugin or theme being used locally
4.	Commit the latest version bits into local folder of your cloned repo
5.	Push your changes to the your forked repo
6.	Login to Azure portal and select your web app
7.	Click on Application Settings -> App Settings and change GIT_REPO to use your repository from step #1. If you haven't changed the branch name, you can continue to use linuxapservice. If you wish to use a different branch, update GIT_BRANCH setting as well.

##### OPTION 2 :
You can update ```WEBSITES_ENABLE_APP_SERVICE_STORAGE``` = true  to enable app service storage to have file persistence. Note when there are issues with storage  due to networking or when app service platform is being updated, your app can be impacted.
You can use below composer cmds to install theme/modules. 

[More Informatio](https://www.drupal.org/docs/develop/using-composer/using-composer-to-manage-drupal-site-dependencies):

```
cd /home/drupal-prj
composer require drupal/redis
composer require drupal/adminimal_theme
```
## Limitations
- Must include  App Setting ```WEBSITES_ENABLE_APP_SERVICE_STORAGE``` = true  as soon as you need files to be persisted.
- Deploy to Azure, Pull and run this image need some time, You can include App Setting ```WEBSITES_CONTAINER_START_TIME_LIMIT``` to specify the time in seconds as need, Default is 240 and max is 1800, suggest to set it as 900 when using this version.

## Change Log 
- **php7.2.9-varnish**
  1. Add Varnish, improve performance.  
- **php7.2.9**
  1. Upgrade php version.
  2. Include composer require-dev.
  3. Required drupal/redis and predis/predis by composer.
  4. Use 'Git pull' to get drupal project codes, the cost time is much shorter than 'composer create-project' in Azure.
  5. Add selectable listen type of php-fpm/nginx.  
- **Version 0.43-composer**
  1. Use "composer create-project" to download latest drupal core.  [More Informatio](https://www.drupal.org/docs/develop/using-composer/using-composer-to-manage-drupal-site-dependencies)
  2. Update composer by entrypoint.sh, always keep it as latest.  
- **Version 0.43**
  1. Installed php extension redis, and local redis-server.
  2. Fix the bug of Drush.
- **Version 0.42**
  1. Update settings of opcache, more stable.  
- **Version 0.41**
  1. Reduce size.
  2. Update version php-fpm.
- **Version 0.4**
  1. Base image to alpine, reduce size.
  2. Update version of nginx and php-fpm.
  3. Update conf files of php-fpm, pass env parameters by default.
  4. Update conf files of nignx.   
- **Version 0.31**
  1. Install some common debug tools, netstat, tcpping, tcpdump.
- **Version 0.3**
  1. Use Git to deploy Drupal.
  2. Add Xdebug extension of PHP.
  3. Update version of nginx to 1.13.11.
  4. Update version of phpmyadmin to 4.8.0.
- **Version 0.2**
  1. Supports local MySQL.
  2. Create default database - azurelocaldb.(You need set DATABASE_TYPE to **"local"**)
  3. Considering security, please set database authentication info on [*"App settings"*](#How-to-configure-to-use-Local-Database-with-web-app) when enable **"local"** mode.
     Note: the credentials below is also used by phpMyAdmin.
      -  DATABASE_USERNAME | <*your phpMyAdmin user*>
      -  DATABASE_PASSWORD | <*your phpMyAdmin password*>
  4. Fixed Restart block issue.

# How to Contribute
If you have feedback please create an issue but **do not send Pull requests** to these images since any changes to the images needs to tested before it is pushed to production.