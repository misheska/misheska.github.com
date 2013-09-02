---
layout: post
title: "Getting Started Writing Chef Cookbooks the Berkshelf Way, Part 2"
date: 2013-06-23 11:37
comments: true
categories: chef
---
* list element with functor item
{:toc}

_Updated September 1, 2013_

* _Bumped apache2 cookbook reference from 1.6.x to 1.7.x_
* _Bumped database cookbook reference from 1.3.x to 1.4.x_

_Updated August 7, 2013_

* _Fixed error in Iteration #10 test per Jeff Thomas_

_Updated July 23rd, 2013_

* _Referenced Sean OMeara's & Charles Johnson's latest myface example app_

This is a second article in a series on writing Opscode Chef cookbooks the
Berkshelf Way.  Here's a link to [Part 1](http://misheska.com/blog/2013/06/16/getting-started-writing-chef-cookbooks-the-berkshelf-way/).  The source code
examples covered in this article can be found on Github:
<https://github.com/misheska/myface>

In this installment, [Part 2](http://misheska.com/blog/2013/06/23/getting-started-writing-chef-cookbooks-the-berkshelf-way-part2/),
 we're going to create a new `database` recipe.  In [Part1](http://misheska.com/blog/2013/06/16/getting-started-writing-chef-cookbooks-the-berkshelf-way/)
MyFace is just a web application serving up a static page.  Now we're going to
enhance MyFace so that it stores account information in a persistent 
MySQL database.

Thanks go out to the Opscode Advanced Chef Cookbook Authoring class and specifically 
[Sean OMeara and Charles Johnson](https://github.com/someara/myface-cookbook) for the database and PHP code used in this article.

Iteration #7 - Install MySQL
============================

Edit `metadata.rb` and add a reference to the `mysql` cookbook.  Also
bump the version to 2.0.0 because we know that there will be incompatible API
changes, moving to MySQL, per [Semantic Versioning](http://semver.org/):

{% codeblock myface/metadata.rb lang:ruby %}
name             'myface'
maintainer       'Mischa Taylor'
maintainer_email 'mischa@misheska.com'
license          'Apache 2.0'
description      'Installs/Configures myface'
long_description IO.read(File.join(File.dirname(__FILE__), 'README.md'))
version          '2.0.0'

depends "apache2", "~> 1.7.0"
depends "mysql", "~> 3.0.0"
{% endcodeblock %}

Create a new recipe called `recipes/database.rb` which includes the MySQL
cookbook's server recipe (this is a similar abstraction to what you created in
[Part 1](http://misheska.com/blog/2013/06/16/getting-started-writing-chef-cookbooks-the-berkshelf-way/)
with `recipes/webserver.rb`):

{% codeblock myface/recipes/database.rb lang:ruby %}
#
# Cookbook Name:: myface
# Recipe:: database
#
# Copyright (C) 2012 YOUR_NAME
# 
# All rights reserved - Do Not Redistribute
#

include_recipe "mysql::server"
{% endcodeblock %}

Wire the `database` recipe into the MyFace cookbook by adding an
`include_recipe` reference to `recipes/default.rb`:

{% codeblock myface/recipes/default.rb lang:ruby %}
# Cookbook Name:: myface
# Recipe:: default
#
# Copyright (C) 2013 YOUR_NAME
#
# All rights reserved - Do Not Redistribute
#

include_recipe "myface::database"
include_recipe "myface::webserver"
{% endcodeblock %}

Run `vagrant provision` to converge your changes.

    $ vagrant provision
    [Berkshelf] Updating Vagrant's berkshelf: '/Users/misheska/.berkshelf/default/vagrant/berkshelf-20130807-5122-1ipka2m-default'
    [Berkshelf] Using myface (1.0.0)
    [Berkshelf] Using apache2 (1.7.0)
    [Berkshelf] Using mysql (3.0.4)
    [Berkshelf] Using openssl (1.1.0)
    [Berkshelf] Using build-essential (1.4.2)
    [default] Chef 11.6.0 Omnibus package is already installed.
    [default] Running provisioner: chef_solo...
    Generating chef JSON and uploading...
    Running chef-solo...
    [2013-08-07T21:18:50-07:00] INFO: Forking chef instance to converge...
    [2013-08-07T21:18:50-07:00] INFO: *** Chef 11.6.0 ***
    [2013-08-07T21:18:50-07:00] INFO: Setting the run_list to ["recipe[myface::default]"] from JSON
    [2013-08-07T21:18:50-07:00] INFO: Run List is [recipe[myface::default]]
    [2013-08-07T21:18:50-07:00] INFO: Run List expands to [myface::default]
    [2013-08-07T21:18:50-07:00] INFO: Starting Chef Run for myface-berkshelf
    [2013-08-07T21:18:50-07:00] INFO: Running start handlers
    [2013-08-07T21:18:50-07:00] INFO: Start handlers complete.
    [2013-08-07T21:18:51-07:00] WARN: Cloning resource attributes for directory[/var/lib/mysql] from prior resource (CHEF-3694)
    [2013-08-07T21:18:51-07:00] WARN: Previous directory[/var/lib/mysql]: /tmp/vagrant-chef-1/chef-solo-1/cookbooks/mysql/recipes/server.rb:108:in `block in from_file'
    [2013-08-07T21:18:51-07:00] WARN: Current  directory[/var/lib/mysql]: /tmp/vagrant-chef-1/chef-solo-1/cookbooks/mysql/recipes/server.rb:108:in `block in from_file'
    [2013-08-07T21:18:51-07:00] INFO: Could not find previously defined grants.sql resource
    [2013-08-07T21:18:51-07:00] WARN: Cloning resource attributes for service[mysql] from prior resource (CHEF-3694)
    [2013-08-07T21:18:51-07:00] WARN: Previous service[mysql]: /tmp/vagrant-chef-1/chef-solo-1/cookbooks/mysql/recipes/server.rb:154:in `from_file'
    [2013-08-07T21:18:51-07:00] WARN: Current  service[mysql]: /tmp/vagrant-chef-1/chef-solo-1/cookbooks/mysql/recipes/server.rb:218:in `from_file'
    [2013-08-07T21:18:51-07:00] WARN: Cloning resource attributes for service[apache2] from prior resource (CHEF-3694)
    [2013-08-07T21:18:51-07:00] WARN: Previous service[apache2]: /tmp/vagrant-chef-1/chef-solo-1/cookbooks/apache2/recipes/default.rb:24:in `from_file'
    [2013-08-07T21:18:51-07:00] WARN: Current  service[apache2]: /tmp/vagrant-chef-1/chef-solo-1/cookbooks/apache2/recipes/default.rb:221:in `from_file'
    [2013-08-07T21:18:52-07:00] INFO: package[mysql] installing mysql-5.1.69-1.el6_4 from updates repository
    [2013-08-07T21:19:01-07:00] INFO: package[mysql-devel] installing mysql-devel-5.1.69-1.el6_4 from updates repository
    [2013-08-07T21:19:24-07:00] INFO: package[mysql-server] installing mysql-server-5.1.69-1.el6_4 from updates repository
    [2013-08-07T21:19:42-07:00] INFO: package[mysql-server] sending start action to service[mysql] (immediate)
    [2013-08-07T21:19:43-07:00] INFO: service[mysql] started
    [2013-08-07T21:19:43-07:00] INFO: directory[/var/log/mysql] created directory /var/log/mysql
    [2013-08-07T21:19:43-07:00] INFO: directory[/var/log/mysql] owner changed to 27
    [2013-08-07T21:19:43-07:00] INFO: directory[/var/log/mysql] group changed to 27
    [2013-08-07T21:19:43-07:00] INFO: directory[/etc] owner changed to 27
    [2013-08-07T21:19:43-07:00] INFO: directory[/etc] group changed to 27
    [2013-08-07T21:19:43-07:00] INFO: directory[/etc/mysql/conf.d] created directory /etc/mysql/conf.d
    [2013-08-07T21:19:43-07:00] INFO: directory[/etc/mysql/conf.d] owner changed to 27
    [2013-08-07T21:19:43-07:00] INFO: directory[/etc/mysql/conf.d] group changed to 27
    [2013-08-07T21:19:43-07:00] INFO: service[mysql] enabled
    [2013-08-07T21:19:43-07:00] INFO: execute[assign-root-password] ran successfully
    [2013-08-07T21:19:43-07:00] INFO: template[/etc/mysql_grants.sql] created file /etc/mysql_grants.sql
    [2013-08-07T21:19:43-07:00] INFO: template[/etc/mysql_grants.sql] updated file contents /etc/mysql_grants.sql
    [2013-08-07T21:19:43-07:00] INFO: template[/etc/mysql_grants.sql] owner changed to 0
    [2013-08-07T21:19:43-07:00] INFO: template[/etc/mysql_grants.sql] group changed to 0
    [2013-08-07T21:19:43-07:00] INFO: template[/etc/mysql_grants.sql] mode changed to 600
    [2013-08-07T21:19:43-07:00] INFO: template[/etc/mysql_grants.sql] sending run action to execute[mysql-install-privileges] (immediate)
    [2013-08-07T21:19:43-07:00] INFO: execute[mysql-install-privileges] ran successfully
    [2013-08-07T21:19:43-07:00] INFO: template[/etc/my.cnf] backed up to /var/chef/backup/etc/my.cnf.chef-20130807211943
    [2013-08-07T21:19:43-07:00] INFO: template[/etc/my.cnf] updated file contents /etc/my.cnf
    [2013-08-07T21:19:43-07:00] INFO: template[/etc/my.cnf] sending restart action to service[mysql] (immediate)
    [2013-08-07T21:19:49-07:00] INFO: service[mysql] restarted
    [2013-08-07T21:19:50-07:00] INFO: Chef Run complete in 59.147486543 seconds
    [2013-08-07T21:19:50-07:00] INFO: Running report handlers
    [2013-08-07T21:19:50-07:00] INFO: Report handlers complete

Testing Iteration #7
--------------------
Verify that the `mysqld` service is running on your vagrant guest by
running the following command:

    $ vagrant ssh -c "sudo /sbin/service mysqld status"
    mysqld (pid  8525) is running...

Also check that MySQL is enabled to start on boot:

    $ vagrant ssh -c "sudo /sbin/chkconfig --list | grep mysqld"
    mysqld         	0:off	1:off	2:on	3:on	4:on	5:on	6:off

If the service is set to be activated at runlevels 3 and 5, then MySQL is
enabled to run under full multi-user text mode and full multi-user graphical
mode, which is exactly the desired behavior.

Iteration #8 - Create the MyFace Database
=========================================

We've installed MySQL, but we don't have a database yet.  Now we're going
to create a database to store information about our users with another
cookbook, the `database` cookbook.

Add the `database` cookbook as a dependency in the `metadata.rb` file:

{% codeblock myface/metadata.rb lang:ruby %}
name             "myface"
maintainer       "Mischa Taylor"
maintainer_email "mischa@misheska.com"
license          "Apache 2.0"
description      "Installs/Configures myface"
long_description IO.read(File.join(File.dirname(__FILE__), 'README.md'))
version          '2.0.0'

depends "apache2", "~> 1.7.0"
depends "mysql", "~> 3.0.0"
depends "database", "~> 1.4.0"
{% endcodeblock %}

Berkshelf automatically populates MySQL passwords for you.  They were
configured in the `Vagrantfile` when you ran `berks cookbook` in
[Part 1](http://misheska.com/blog/2013/06/16/getting-started-writing-chef-cookbooks-the-berkshelf-way/#create-the-myface-application-cookbook):

    ...
    config.vm.provision :chef_solo do |chef|
      chef.json = {
        :mysql => {
          :server_root_password => 'rootpass',
          :server_debian_password => 'debpass',
          :server_repl_password => "replpass'
        }
      }
      ...
    end
    ...

You can reference these passwords as variables in your Chef recipes, which we
will do when we add some data attributes.  Add the following attributes to
`attributes/default.rb` so it looks like so:

{% codeblock myface/attributes/default.rb lang:ruby %}
default[:myface][:user] = "myface"
default[:myface][:group] = "myface"
default[:myface][:name] = "myface"
default[:myface][:config] = "myface.conf"
default[:myface][:document_root] = "/srv/apache/myface"

default[:myface][:database][:host] = 'localhost'
default[:myface][:database][:username] = 'root'
default[:myface][:database][:password] = node[:mysql][:server_root_password]
default[:myface][:database][:dbname] = 'myface'
{% endcodeblock %}

Describe the database to be created for MyFace in `recipes/database.rb`:

{% codeblock myface/recipes/database.rb lang:ruby %}
#
# Cookbook Name:: myface
# Recipe:: database
#
# Copyright (C) 2012 YOUR_NAME
#
# All rights reserved - Do Not Redistribute
#

include_recipe "mysql::server"
include_recipe "database::mysql"

mysql_database node[:myface][:database][:dbname] do
  connection(
    :host => node[:myface][:database][:host],
    :username => node[:myface][:database][:username],
    :password => node[:myface][:database][:password]
  )
  action :create
end
{% endcodeblock %}

Converge the changes with `vagrant provision`:

    $ vagrant provision
    [Berkshelf] Updating Vagrant's berkshelf: '/Users/misheska/.berkshelf/default/vagrant/berkshelf-20130807-5122-1ipka2m-default'
    [Berkshelf] Using myface (2.0.0)
    [Berkshelf] Using apache2 (1.7.0)
    [Berkshelf] Using mysql (3.0.4)
    [Berkshelf] Using openssl (1.1.0)
    [Berkshelf] Using build-essential (1.4.2)
    [Berkshelf] Installing database (1.4.0) from site: 'http://cookbooks.opscode.com/api/v1/cookbooks'
    [Berkshelf] Installing postgresql (3.0.4) from site: 'http://cookbooks.opscode.com/api/v1/cookbooks'
    [Berkshelf] Installing apt (2.1.1) from site: 'http://cookbooks.opscode.com/api/v1/cookbooks'
    [Berkshelf] Installing aws (0.101.4) from site: 'http://cookbooks.opscode.com/api/v1/cookbooks'
    [Berkshelf] Installing xfs (1.1.0) from site: 'http://cookbooks.opscode.com/api/v1/cookbooks'
    [default] Chef 11.6.0 Omnibus package is already installed.
    [default] Running provisioner: chef_solo...
    Generating chef JSON and uploading...
    Running chef-solo...
    [2013-08-07T21:26:11-07:00] INFO: Forking chef instance to converge...
    [2013-08-07T21:26:11-07:00] INFO: *** Chef 11.6.0 ***
    [2013-08-07T21:26:12-07:00] INFO: Setting the run_list to ["recipe[myface::default]"] from JSON
    [2013-08-07T21:26:12-07:00] INFO: Run List is [recipe[myface::default]]
    [2013-08-07T21:26:12-07:00] INFO: Run List expands to [myface::default]
    [2013-08-07T21:26:12-07:00] INFO: Starting Chef Run for myface-berkshelf
    [2013-08-07T21:26:12-07:00] INFO: Running start handlers
    [2013-08-07T21:26:12-07:00] INFO: Start handlers complete.
    [2013-08-07T21:26:12-07:00] WARN: Cloning resource attributes for directory[/var/lib/mysql] from prior resource (CHEF-3694)
    [2013-08-07T21:26:12-07:00] WARN: Previous directory[/var/lib/mysql]: /tmp/vagrant-chef-1/chef-solo-1/cookbooks/mysql/recipes/server.rb:108:in `block in from_file'
    [2013-08-07T21:26:12-07:00] WARN: Current  directory[/var/lib/mysql]: /tmp/vagrant-chef-1/chef-solo-1/cookbooks/mysql/recipes/server.rb:108:in `block in from_file'
    [2013-08-07T21:26:12-07:00] INFO: Could not find previously defined grants.sql resource
    [2013-08-07T21:26:12-07:00] WARN: Cloning resource attributes for service[mysql] from prior resource (CHEF-3694)
    [2013-08-07T21:26:12-07:00] WARN: Previous service[mysql]: /tmp/vagrant-chef-1/chef-solo-1/cookbooks/mysql/recipes/server.rb:154:in `from_file'
    [2013-08-07T21:26:12-07:00] WARN: Current  service[mysql]: /tmp/vagrant-chef-1/chef-solo-1/cookbooks/mysql/recipes/server.rb:218:in `from_file'
    [2013-08-07T21:26:14-07:00] INFO: package[autoconf] installing autoconf-2.63-5.1.el6 from base repository
    [2013-08-07T21:26:22-07:00] INFO: package[bison] installing bison-2.4.1-5.el6 from base repository
    [2013-08-07T21:26:29-07:00] INFO: package[flex] installing flex-2.5.35-8.el6 from base repository
    [2013-08-07T21:26:36-07:00] INFO: package[gcc-c++] installing gcc-c++-4.4.7-3.el6 from base repository
    [2013-08-07T21:27:30-07:00] WARN: Cloning resource attributes for service[apache2] from prior resource (CHEF-3694)
    [2013-08-07T21:27:30-07:00] WARN: Previous service[apache2]: /tmp/vagrant-chef-1/chef-solo-1/cookbooks/apache2/recipes/default.rb:24:in `from_file'
    [2013-08-07T21:27:30-07:00] WARN: Current  service[apache2]: /tmp/vagrant-chef-1/chef-solo-1/cookbooks/apache2/recipes/default.rb:221:in `from_file'
    [2013-08-07T21:27:31-07:00] INFO: Chef Run complete in 78.857442041 seconds
    [2013-08-07T21:27:31-07:00] INFO: Running report handlers
    [2013-08-07T21:27:31-07:00] INFO: Report handlers complete

Testing Iteration #8
--------------------
Run `mysqlshow` on your vagrant guest to display database information, verifying
that the `myface` database was created:

    $ vagrant ssh -c "mysqlshow -uroot -prootpass"
    +--------------------+
    |     Databases      |
    +--------------------+
    | information_schema |
    | myface             |
    | mysql              |
    | test               |
    +--------------------+

Note that `myface` is listed as a database name - success!

Iteration #9 - Create a MySQL user
==================================

It's a good idea to create a user in MySQL for each one of your applications
that has the ability to only manipulate the application's database and has 
no MySQL administrative privileges.

Add some attributes to `attributes/default.rb` for your app user:

    default[:myface][:database][:app][:username] = 'myface_app'
    default[:myface][:database][:app][:password] = 'supersecret'

`attributes/default.rb` should look like so:
{% codeblock myface/attributes/default.rb lang:ruby %}
default[:myface][:user] = "myface"
default[:myface][:group] = "myface"
default[:myface][:name] = "myface"
default[:myface][:config] = "myface.conf"
default[:myface][:document_root] = "/srv/apache/myface"

default[:myface][:database][:host] = 'localhost'
default[:myface][:database][:username] = 'root'
default[:myface][:database][:password] = node[:mysql][:server_root_password]
default[:myface][:database][:dbname] = 'myface'

default[:myface][:database][:app][:username] = 'myface_app'
default[:myface][:database][:app][:password] = 'supersecret'
{% endcodeblock %}


Edit `recipes/database.rb` and describe the MySQL database user:

    ...
      )
      action :create
    end

    mysql_database_user node[:myface][:database][:app][:username] do
      connection(
        :host => node[:myface][:database][:host],
        :username => node[:myface][:database][:username],
        :password => node[:myface][:database][:password]
      )
      password node[:myface][:database][:app][:password]
      database_name node[:myface][:database][:dbname]
      host default[:myface][:database][:host]
      action [:create, :grant]
    end

After editing `recipes/database.rb` should look like the following:

{% codeblock myface/recipes/database.rb lang:ruby %}
#
# Cookbook Name:: myface
# Recipe:: database
#
# Copyright (C) 2012 YOUR_NAME
#
# All rights reserved - Do Not Redistribute
#

include_recipe "mysql::server"
include_recipe "database::mysql"

mysql_database node[:myface][:database][:dbname] do
  connection(
    :host => node[:myface][:database][:host],
    :username => node[:myface][:database][:username],
    :password => node[:myface][:database][:password]
  )
  action :create
end

mysql_database_user node[:myface][:database][:app][:username] do
  connection(
    :host => node[:myface][:database][:host],
    :username => node[:myface][:database][:username],
    :password => node[:myface][:database][:password]
  )
  password node[:myface][:database][:app][:password]
  database_name node[:myface][:database][:dbname]
  host node[:myface][:database][:host]
  action [:create, :grant]
end
{% endcodeblock %}

Converge the node to apply the changes:

    $ vagrant provision

Testing Iteration #9
--------------------

Check to see if the `myface-app` user is enabled as a local user by running the
following mysql command:

    $ vagrant ssh -c 'mysql -uroot -prootpass -e "select user,host from mysql.user;"'
    user	  host
    repl	  %
    root	  127.0.0.1
         	  localhost
    myface	  localhost
    myface_app    localhost
    root	  localhost
        	   myface-berkshelf

As you can see above, the `myface_app@localhost` user exists, so our cookbook
did what was expected.

Also check to see that the `myface_app` user only has rights on the myface databse:

    $ vagrant ssh -c 'mysql -uroot -prootpass -e "show grants for 'myface_app'@'localhost';"'
    Grants for myface_app@localhost
    GRANT USAGE ON *.* TO 'myface_app'@'localhost' IDENTIFIED BY PASSWORD '*90BA3AC0BFDE07AE334CA523CB27167AE33825B9'
    GRANT ALL PRIVILEGES ON `myface`.* TO 'myface_app'@'localhost'

Iteration #10 - Create a table for users
========================================

Let's create a SQL script to create a table modeling MyFace users and
populate it with some initial data.  Create a file 
`files/default/myface-create.sql` with the following content:

{% codeblock myface/files/default/myface-create.sql lang:sql %}
/* A table for myface users */
 
CREATE TABLE users(
 id CHAR (32) NOT NULL,
 PRIMARY KEY(id),
 user_name VARCHAR(64),
 url VARCHAR(256),
 email VARCHAR(128),
 neck_beard INTEGER
);
 
/* Initial records */
INSERT INTO users ( id, user_name, url, email, neck_beard ) VALUES ( uuid(), 'jtimberman', 'http://jtimberman.housepub.org', 'joshua@opscode.com', 4 );
INSERT INTO users ( id, user_name, url, email, neck_beard ) VALUES ( uuid(), 'someara', 'http://blog.afistfulofservers.net/', 'someara@opscode.com', 5 );
INSERT INTO users ( id, user_name, url, email, neck_beard ) VALUES ( uuid(), 'jwinsor', 'http://vialstudios.com', 'jamie@vialstudios.com', 4 );
INSERT INTO users ( id, user_name, url, email, neck_beard ) VALUES ( uuid(), 'cjohnson', 'http://www.chipadeedoodah.com/', 'charles@opscode.com', 3 );
INSERT INTO users ( id, user_name, url, email, neck_beard ) VALUES ( uuid(), 'mbower', 'http://www.webbower.com/', 'matt@webbower.com', 4 );
{% endcodeblock %}

Add an attribute for the temporary location used for the SQL script we just
created:

    default[:myface][:database][:seed_file] = '/tmp/myface-create.sql'

The resultant `attributes/default.rb` should resemble the following:
{% codeblock myface/attributes/default.rb lang:ruby %}
default[:myface][:user] = "myface"
default[:myface][:group] = "myface"
default[:myface][:name] = "myface"
default[:myface][:config] = "myface.conf"
default[:myface][:document_root] = "/srv/apache/myface"

default[:myface][:database][:host] = 'localhost'
default[:myface][:database][:username] = 'root'
default[:myface][:database][:password] = node[:mysql][:server_root_password]
default[:myface][:database][:dbname] = 'myface'

default[:myface][:database][:app][:username] = 'myface_app'
default[:myface][:database][:app][:password] = 'supersecret'

default[:myface][:database][:seed_file] = '/tmp/myface-create.sql'
{% endcodeblock %}

Modify `recipes/database.rb` so that the cookbook transfers the SQL script
to the guest node and so that the SQL script executes.  As you learned in
Part 1, [recipes should be idempotent](http://misheska.com/blog/2013/06/16/getting-started-writing-chef-cookbooks-the-berkshelf-way/#testing-iteration-1),
so you will need to add a `not_if` statement which ensures that the command
is only executed when necessary.

    ...
      host "localhost"
      action [:create, :grant]
    end

    # Write schema seed file to filesystem
    cookbook_file node[:myface][:database][:seed_file] do
      source "myface-init.sql"
      owner "root"
      group "root"
      mode "0600"
    end

    # Seed database with test data
    execute "initialize myface database" do
      command "mysql -h #{node[:myface][:database][:host]} -u #{node[:myface][:database][:app][:username]} -p#{node[:myface][:database][:app][:password]} -D #{node[:myface][:database][:dbname]} < #{node[:myface][:database][:seed_file]}"
      not_if  "mysql -h #{node[:myface][:database][:host]} -u #{node[:myface][:database][:app][:username]} -p#{node[:myface][:database][:app][:password]} -D #{node[:myface][:database][:dbname]} -e 'describe users;'"
    end

Once you have made these changes, `recipes/database.rb` should look like so:

{% codeblock myface/recipes/database.rb lang:ruby %}
#
# Cookbook Name:: myface
# Recipe:: database
#
# Copyright (C) 2012 YOUR_NAME
#
# All rights reserved - Do Not Redistribute
#

include_recipe "mysql::server"
include_recipe "database::mysql"

mysql_database node[:myface][:database][:dbname] do
  connection(
    :host => node[:myface][:database][:host],
    :username => node[:myface][:database][:username],
    :password => node[:myface][:database][:password]
  )
  action :create
end

mysql_database_user node[:myface][:database][:app][:username] do
  connection(
    :host => node[:myface][:database][:host],
    :username => node[:myface][:database][:username],
    :password => node[:myface][:database][:password]
  )
  password node[:myface][:database][:app][:password]
  database_name node[:myface][:database][:dbname]
  host node[:myface][:database][:host]
  action [:create, :grant]
end

# Write schema seed file to filesystem
cookbook_file node[:myface][:database][:seed_file] do
  source "myface-create.sql"
  owner "root"
  group "root"
  mode "0600"
end

# Seed database with test data
execute "initialize myface database" do
  command "mysql -h #{node[:myface][:database][:host]} -u #{node[:myface][:database][:app][:username]} -p#{node[:myface][:database][:app][:password]} -D #{node[:myface][:database][:dbname]} < #{node[:myface][:database][:seed_file]}"
  not_if  "mysql -h #{node[:myface][:database][:host]} -u #{node[:myface][:database][:app][:username]} -p#{node[:myface][:database][:app][:password]} -D #{node[:myface][:database][:dbname]} -e 'describe users;'"
end
{% endcodeblock %}

Run `vagrant provision` to converge your changes:

    $ vagrant provision

Testing Iteration #10
---------------------

Run the following mysql command to dump the contents of the `users` table:

    $ vagrant ssh -c 'mysql -hlocalhost -umyface_app -psupersecret -Dmyface -e "select id,user_name from users;"'
    id                                  user_name
    216e03c2-ffe4-11e2-b1ad-080027c8	jtimberman
    216e0890-ffe4-11e2-b1ad-080027c8	someara
    216e0bce-ffe4-11e2-b1ad-080027c8	jwinsor
    216e0eda-ffe4-11e2-b1ad-080027c8	cjohnson
    216e11e6-ffe4-11e2-b1ad-080027c8	mbower

The output should look similar to what you see above - the data from the
`INSERT INTO` statemens in the SQL script.

Iteration #11 - Install PHP
===========================

Let's add some PHP scripting sizzle to sell the steak of the database we
just created.  We're going to install Apache 2 `mod_php5` module and the
`php-mysql` package to support our PHP script.

Edit `recipes/webserver.rb` and add the following:

    ...
    include_recipe "apache2"
    include_recipe "apache2::mod_php5"

    package "php-mysql" do
      action :install
      notifies :restart, "service[apache2]"
    end

This will use the apache2 cookbook's `mod_php5` module to install PHP5
and install the `php-mysql` support package.  After editing,
`recipes/webserver.rb` should look like this:

{% codeblock myface/recipes/webserver.rb lang:ruby %}
#
# Cookbook Name:: myface
# Recipe:: webserver
#
# Copyright (C) 2013 YOUR_NAME
#
# All rights reserved - Do Not Redistribute
#

group node[:myface][:group]

user node[:myface][:user] do
  group node[:myface][:group]
  system true
  shell "/bin/bash"
end

include_recipe "apache2"
include_recipe "apache2::mod_php5"

package "php-mysql" do
  action :install
  notifies :restart, "service[apache2]"
end

# disable default site
apache_site "000-default" do
  enable false
end

# create apache config
template "#{node[:apache][:dir]}/sites-available/#{node[:myface][:config]}" do
  source "apache2.conf.erb"
  notifies :restart, 'service[apache2]'
end

# create document root
directory "#{node[:myface][:document_root]}" do
  action :create
  recursive true
end

# write site
cookbook_file "#{node[:myface][:document_root]}/index.html" do
  mode "0644"
end

# enable myface
apache_site "#{node[:myface][:config]}" do
  enable true
end
{% endcodeblock %}

Run `vagrant provision` to converge your changes:

    $ vagrant provision

Test Iteration #11
------------------

Run the following command to verify that the php5_module was successfully
installed:

    $ vagrant ssh -c "sudo /usr/sbin/httpd -M | grep php5"
    httpd: Could not reliably determine the server's fully qualified domain name, using 127.0.0.1 for ServerName
    [Wed Aug 07 21:42:03 2013] [warn] NameVirtualHost *:80 has no VirtualHosts
    Syntax OK
     php5_module (shared)

Iteration #12 - Add PHP Sizzle
==============================

It's the last iteration, get ready to see the PHP sizzle!  First modify
`templates/default/apache2.conf.erb` as follows:

{% codeblock myface/templates/default/apache2.conf.erb lang:ruby %}
# Managed by Chef for <%= node[:hostname] %>
<VirtualHost *:80>
        ServerAdmin <%= node[:apache][:contact] %>

        DocumentRoot <%= node[:myface][:document_root] %>
        <Directory />
                Options FollowSymLinks
                AllowOverride None
        </Directory>
        <Directory <%= node[:myface][:document_root] %>>
                Options Indexes FollowSymLinks MultiViews
                AllowOverride None
                Order allow,deny
                allow from all
        </Directory>

        ErrorLog <%= node[:apache][:log_dir] %>/error.log

        LogLevel warn

        CustomLog <%= node[:apache][:log_dir] %>/access.log combined
        ServerSignature Off
</VirtualHost>
{% endcodeblock %}

Next delete `files/default/index.html` with the following command:

    $ rm files/default/index.html

You'll be replacing it with the following parametized PHP script as
`templates/default/index.php.erb`:

{% codeblock myface/templates/default/index.php.erb lang:ruby %}
<!--
Copyright 2013, Opscode, Inc.
 
Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at
 
    http://www.apache.org/licenses/LICENSE-2.0
Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
 
-->
 
<?php
$db_host = 'localhost';
$db_user = 'root';
$db_pwd = '<%= node['mysql']['server_root_password'] %>';
 
$database = 'myface';
$table = 'users';
 
// UTILITY FUNCTIONS
function create_gravatar_hash($email) {
	return md5( strtolower( trim( $email ) ) );
}
 
function gravatar_img($email=null, $name=null, $size=null) {
	if(!$email) {
		return '';
	}
	
	$url =  'http://www.gravatar.com/avatar/';
	$url .= create_gravatar_hash($email);
	if($size) {
		$url .= "?s={$size}";
	}
	
	return sprintf('<img src="%s" alt="%s" />', $url, $name ? $name : '');
}
 
function neckbeard($rating) {
	$ratings = array(
		'Puberty awaits!',
		'Peach fuzz',
		'Solid week&#39;s growth',
		'Lumberjacks would be proud',
		'Makes dwarves weep',
	);
	
	return $ratings[(int) $rating - 1];
}
 
// Fire up the database
if (!mysql_connect($db_host, $db_user, $db_pwd))
    die("Can't connect to database");
 
if (!mysql_select_db($database))
    die("Can't select database");
 
// sending query
$result = mysql_query("SELECT * FROM {$table}");
if (!$result) {
    die("Query to show fields from table failed");
}
 
$fields_num = mysql_num_fields($result);
?>
<!DOCTYPE html>
<html lang="en">
<head>
	<title>MyFace Users</title>
	<style>
	* {
		-webkit-box-sizing: border-box;
		-moz-box-sizing: border-box;
		box-sizing: border-box;
	}
	
	html, body {
		margin: 0;
		padding: 0;
	}
	
	html {
		background: #999;
	}
	
	body {
		max-width: 480px;
		margin: 0 auto;
		font-family: Arial, Helvetica, sans-serif;
		color: #222;
		padding: 20px;
		border: 1px solid #666;
		-webkit-box-shadow: 0 0 5px rgba(0,0,0,0.3);
		-moz-box-shadow: 0 0 5px rgba(0,0,0,0.3);
		box-shadow: 0 0 5px rgba(0,0,0,0.3);
		background: #FFF;
	}
		
	a:link {
		text-decoration: none;
		color: #777;
	}
	
	a:hover,
	a:focus {
		text-decoration: underline;
	}
	
	h1 {
		text-align: center;
		margin-top: 0;
	}
		
	h1 span {
		color: #00C;
	}
	
	h2 {
		font-size: 24px;
		line-height: 1.0;
		margin: 0 0 10px;
	}
	
	p {
		font-size: 14px;
		line-height: 18px;
		margin: 10px 0;
	}
	
	p:last-child {
		margin-bottom: 0;
	}
	
	.email {
		display: block;
		overflow: hidden;
		text-overflow: ellipsis;
		white-space: nowrap;
	}
	
	/* Adapted from OOCSS
		* mod object (https://github.com/stubbornella/oocss/blob/master/core/module/mod.css)
		* media object (https://github.com/stubbornella/oocss/blob/master/core/media/media.css)
	*/
	article {
		display: block;
		overflow: hidden;
		margin-bottom: 20px;
		border: 1px solid #CCC;
		background: #EEE;
		-webkit-border-radius: 4px;
		-moz-border-radius: 4px;
		border-radius: 4px;
		-webkit-box-shadow: 1px 1px 1px rgba(0,0,0,0.3) inset;
		-moz-box-shadow: 1px 1px 1px rgba(0,0,0,0.3) inset;
		box-shadow: 1px 1px 1px rgba(0,0,0,0.3) inset;
	}
 
	article .img {
		float: left;
		margin-right: 10px;
	}
	
	article .img img {
		display: block;
	}
	
	article .imgExt {
		float: right;
		margin-left: 10px;
	}
 
	article .bd {
		overflow: hidden;
		padding: 10px 0;
	}
	</style>
</head>
<body>
	<h1>Welcome to My<span>Face</span>!</h1>
 
	<?php while($row = mysql_fetch_object($result)): ?>
	<article>
		<a href="<?php echo $row->url ?>" class="img" target="_blank">
			<?php echo gravatar_img($row->email, $row->user_name, 150) ?>
		</a>
		<div class="bd">
			<h2><?php echo $row->user_name ?></h2>
			<p><a href="<?php echo $row->url ?>" target="_blank" class="email"><?php echo $row->url ?></a></p>
			<p>Neckbeard rating: <?php echo neckbeard($row->neck_beard) ?></p>
		</div>
	</article>
	<?php endwhile; ?>
</body>
</html>
<?php mysql_free_result($result); ?>
{% endcodeblock %}

Finally modify `recipes/webserver.rb` to use `index.php.erb` template to 
generate a new `index.php` document root: 

{% codeblock myface/recipes/webserver.rb lang:ruby %}
#
# Cookbook Name:: myface
# Recipe:: webserver
#
# Copyright (C) 2013 YOUR_NAME
#
# All rights reserved - Do Not Redistribute
#

group node[:myface][:group]

user node[:myface][:user] do
  group node[:myface][:group]
  system true
  shell "/bin/bash"
end

include_recipe "apache2"
include_recipe "apache2::mod_php5"

package "php-mysql" do
  action :install
  notifies :restart, "service[apache2]"
end

# disable default site
apache_site "000-default" do
  enable false
end

# create apache config
template "#{node[:apache][:dir]}/sites-available/#{node[:myface][:config]}" do
  source "apache2.conf.erb"
  notifies :restart, 'service[apache2]'
end

# create document root
directory "#{node[:myface][:document_root]}" do
  action :create
  mode "0755"
  recursive true
end

# write site
template "#{node[:myface][:document_root]}/index.php" do
  source "index.php.erb"
  mode "0644"
end

# enable myface
apache_site "#{node[:myface][:config]}" do
  enable true
end
{% endcodeblock %}

Since we changed the document root and our recipe contains no statements to
remove the old `index.html` document root, we'll need to destroy our vagrant
test node and do a full `vagrant up` again, otherwise if we visit
<http://33.33.33.10> again, we'll just see the old document root:

    $ vagrant destroy -f
    $ vagrant up

Testing Iteration #12
---------------------

Visit <http://33.33.33.10>  Now you should see the lovely new PHP version of
Myface.

![myfacephp](/images/myfacephp.png)

More to Come!
=============

In [Part 3](http://misheska.com/blog/2013/08/06/getting-started-writing-chef-cookbooks-the-berkshelf-way-part3/),
 we'll introduce a new tool `test-kitchen` and show you how to
automate all the tests you've been doing manually to test each iteration.

If you want to see the full source for MyFace, check out the following
GitHub link: <https://github.com/misheska/myface>
