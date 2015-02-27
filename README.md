My testing redmine instance : redmine.mke.ovh

There is different views for different kind of people as executives and developers don't have the same expectation. Credentials are :
- issuer:password
- stakeholder:password
- developer:password
- manager:password


# Introduction

Project management software tools can be : 
- manual tools : excel, post it & black board, ....
- offline softwares : Microsoft project, projectLibre (MP open source equivalent), ...
- online softwares : in house hosting or hosting in the cloud. The choice is all about politics and privacy concern. A list of available tools : Jira, Trello, Redmine, icescrum, Taiga, Open Atrium, Asana, Basecamp, Azendoo. targetProcess, Trac, Phabricator, ganib, twprojec, Agilefal, fogcreek, ...

# In house hosting possibilities

There is a huge choice of project management tools outhere. If we remove proprietary platforms that we need to install in the cloud, the list lose of its weight. In the open source world we have : 
- Taiga : https://tree.taiga.io (very new and still in beta version)
- Redmine : http://www.redmine.org/ 
- OpenProject : https://www.openproject.org/ (fork of chiliProject which is a fork of Redmine)
- Phabricator : http://phabricator.org/ (developers centric)
- Trac : http://trac.edgewall.org/ (similar in features with redmine but redmine has a subproject feature that is very handy)
- Libre plan : http://demo.libreplan.org/ (important learning curve for non technical users.)
- ]project-open[ : http://project-open.net/ (want to be everything from ERP, CRM to PM tool)

In the proprietary world :
- Github : https://github.com/ (developers centric)
- JIRA : https://fr.atlassian.com/software/jira.html (same feature as redmine)
- twprojec : http://twproject.com/  (same spirit than jira)
- Agilefant : http://agilefant.com/ (ressource heavy : need tomcat and other java based software)

# Redmine

Redmine features that are nice to have :
- can be extend with plugins
- workflow tuning
- a user can be part of different groups and those user or groups can have permissions per project.
- gantt chart (quite basic in the out-of-the-box redmine install but way better with a dedicated plugins)
- can create subproject
- Agile and Kanban (with some plugins)

Redmine main drawback compare to JIRA is a plugin might work well under a specific version of redmine but nothing work in the next version. Under redmine 2.5, the plugins I installed work perfectly and everything work smoothly without and lag or whatsoever.

Redmine is a very powerfull tool, however redmine developers are not designer and the default theme don't look modern at all.

# Installation Cookbook on ubuntu server

## Note for ubuntu
- ubuntu package for redmine is very old. We will use Redmine 2.5
- the main theme is based on minimap with some customization. Because the tool was supposed to be used by a wide range of people within the organization, I wanted a nice touch and feel because if it looks dirty, not everyone  will use it on a everyday basis.

## Install

### Components
- mysql
- nginx
- passenger

### Cookbook

#### setup brand new server
```
apt-get update; 
apt-get upgrade; 
apt-get install emacs tree htop tig nano
```

#### Installing rvm (ruby version manager)
```
  su root
```
```
  apt-get install mysql-server libmysqlclient-dev git-core subversion imagemagick libmagickwand-dev libcurl4-openssl-dev
```
```
  gpg --keyserver hkp://keys.gnupg.net --recv-keys D39DC0E3
  curl -L https://get.rvm.io | bash -s stable --ruby=2.0.0
```
Now, you should load the RVM
```
  source /usr/local/rvm/scripts/rvm
  echo '[[ -s "/usr/local/rvm/scripts/rvm" ]] && source "/usr/local/rvm/scripts/rvm"' >> ~/.bashrc
```

#### Installing Phusion Passenger and Nginx
```
gem install passenger --no-ri --no-rdoc
passenger-install-nginx-module
```
Configuring nginx
```
git clone git://github.com/jnstq/rails-nginx-passenger-ubuntu.git
mv rails-nginx-passenger-ubuntu/nginx/nginx /etc/init.d/nginx
chown root:root /etc/init.d/nginx
update-rc.d nginx defaults
```
```
nano /opt/nginx/conf/nginx.conf
```
Replace server section for port 80
```
server {
  listen  80;
  server_name <your_server_domain_name>;
  root /var/data/redmine/public;
  passenger_enabled on;
  client_max_body_size      10m; # Max attachemnt size
}
```
#### Installing Redmine
``` 
mkdir /var/data
 cd /var/data/
 svn co http://svn.redmine.org/redmine/branches/2.5-stable redmine
 cd /var/data/redmine
```
#### Database configuration
```
nano config/database.yml
```
Add following lines
```
production:
  adapter: mysql2
  database: redmine
  host: localhost
  username: redmine
  password: redmine
  encoding: utf8

development:
  adapter: mysql2
  database: redmine 
  host: localhost
  username: redmine
  password: redmine
  encoding: utf8
```
Plugins installation
Unarchive plugins to /plugins/ folder
```
cd /var/data/redmine
gem install bundle
bundle install
```

#### Configuring redmine
Setup redmine folder permissions
```
cd /var/data/redmine
mkdir public/plugin_assets
chown -R www-data:www-data files log tmp public/plugin_assets config.ru
chmod -R 755 files log tmp public/plugin_assets
```

#### Create database
```
mysql -u root -p
```
Execute following lines to MySQL
```
CREATE DATABASE redmine character SET utf8;
CREATE user 'redmine'@'localhost' IDENTIFIED BY 'redmine';
GRANT ALL privileges ON redmine.* TO 'redmine'@'localhost';
exit
```

#### Migrate database
```
cd /var/data/redmine
bundle exec rake db:migrate
bundle exec rake redmine:plugins
Generate session store
bundle exec rake generate_secret_token 
```
Start web server
```
service nginx start
```
Restart Redmine
```
touch /var/data/redmine/tmp/restart.txt
```

#### References
this cookbook was taken and adapt from :
- Redmine installation cookbook : http://www.redmine.org/projects/redmine/wiki/FrRedmineInstall
- http://www.redminecrm.com/boards/4/topics/448-installing-redmine-2-5-passenger-nginx-rvm-on-ubuntu-12-04-lts-and-14-04-lts


## Redmine tips

### Plugin install/uninstall

Plugin installation may add table to the database. Uninstall a plugin would require to remove some tables + the plugin file. However, it did not work well with some wild plugins I tried.

Backing up the database is important before installation something so that we can properly revert the platform if the plugin don't fit our needs.

- Installation :
```
cd /var/data/redmine/plugins/
git clone $MY_PLUGIN_URL
rake redmine:plugins:migrate RAILS_ENV=production # create required database
service nginx restart # restart server
```
You should now be able to see the plugin list in Administration -> Plugins


- Uninstall :
```
cd /var/data/redmine/plugins/
rake redmine:plugins:migrate NAME=$PLUGIN_NAME VERSION=0 RAILS_ENV=production
service nginx restart # restart server
mkdir ~/.Trash
mv $PLUGIN_NAME ~/.Trash/
```
If an error occured, reload the previous version of your database that you were using without the plugin.

### Backup

#### Database
- Create backup
```
mysqldump -u redmine -p redmine > `date +%y_%m_%d`_redmine.backup.sql
```
- restore database from backup
```
mysql -u redmine -p redmine < ________redmine.backup.sql
```

#### Attachments
- Create backup
```
rsync -a /path/to/redmine/files /path/to/backup/files
```
- Restore from backup
```

```
