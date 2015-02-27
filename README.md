My testing redmine instance : redmine.mke.ovh

There is different view for different kind of people. Credentials are :
- issuer : issuer:password
- stakeholder : stakeholder:password
- developer : developer:password
- manager : manager:pssword


# Introduction

Project management software tools can be : 
- manual tools : excel, post it & black board, ....
- offline softwares : Microsoft project, projectLibre (MP open source equivalent), ...
- online softwares : in house hosting or hosting in the cloud. The choice is all about politics and privacy concern. A list of available tools : Jira, Trello, Redmine, icescrum, Taiga, Open Atrium, Asana, Basecamp, Azendoo. targetProcess, Trac, Phabricator, ganib, twprojec, Agilefal, fogcreek, ...

# In house hosting possibilities.

There is a huge choice of project management tools outhere. If we remove proprietary platforms that we need to install in the cloud, remaining choices are way less important. In the open source world we have : 
- Taiga : https://tree.taiga.io (very new and still in beta version)
- Redmine : http://www.redmine.org/
- OpenProject : https://www.openproject.org/ (fork of chiliProject which is a fork of Redmine)
- Phabricator : http://phabricator.org/ (developers centric)
- Trac : http://trac.edgewall.org/
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
- gantt chart (quite basic in the out-of-the-box redmine install but way better with some plugins)
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


## Redmine tips

### Plugin install/uninstall

Plugin installation may add table to the database. Uninstall a plugin would require to remove some tables + the plugin file. However, it did not work well with some wild plugins I tried.

Backing up the database is important before installation something so that we can properly revert the platform if the plugin don't fit our needs.

Installation :

cd /var/data/redmine/plugins/
git clone $PLUGIN_URL

Uninstall :


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
