Play Framework 2 application on OpenShift Express
=================================================

**WARNING: this quickstrt is in the middle of updating and not yet ready to be used. The plan is to complete it in at the beginning of December 2014.**

This git repository will help you get up and running quickly with a Play Framework 2 application
on OpenShift Express taking advantage of the do-it-yourself cartridge.

TODO: add info about supported versions/where to go for 2.0.x/2.1.x

TODO: add info about tested versions

TODO: add info about `play` command deprecation


Running on OpenShift
--------------------

Create a new Play Framework 2 application:

    activator new play2demo
    cd play2demo

    git init

Register at https://openshift.redhat.com/, and then create a diy (do-it-yourself) application:

    rhc app create play2demo -t diy-0.1 --no-git -l yourlogin

You will see something like the following:


```bash
Application Options
-------------------
Domain:     yourdomain
Cartridges: diy-0.1
Gear Size:  default
Scaling:    no

Creating application 'play2demo' ... done

  Disclaimer: This is an experimental cartridge that provides a way to try unsupported languages, frameworks, and middleware on OpenShift.

Waiting for your DNS name to be available ... done

Your application 'play2demo' is now available.

  URL:        http://play2demo-yourdomain.rhcloud.com/
  SSH to:     your_uuid@play2demo-yourdomain.rhcloud.com
  Git remote: ssh://your_uuid@play2demo-yourdomain.rhcloud.com/~/git/play2demo.git/

Run 'rhc show-app play2demo' for more details about your app.
```

Copy and paste the git url to add it as a remote repo (replace the `uuid` part with your own!):

    git remote add origin ssh://your_uuid@play2demo-yourdomain.rhcloud.com/~/git/play2demo.git/
    git pull -s recursive -X theirs origin master
    git add .
    git commit -m "initial deploy"

And then add this repository as a remote repo named quickstart:

    git remote add quickstart -m master git://github.com/EugenyLoy/play2-openshift-quickstart.git
    git pull -s recursive -X theirs quickstart master


Then use the stage task to prepare your deployment:

    activator clean stage

And add your changes to git's index, commit and push the repo upstream:

    git add .
    git commit -m "a nice message"
    git push origin

That's it, you can now see your application running at:

    http://play2demo-yourdomain.rhcloud.com

The first time you do it, it will take quite a few minutes to complete, because git has to upload Play's dependencies, but after that git is smart enough to just upload the differences.

To deploy your changes, you can just repeat the steps from `activator clean stage`, or use the helper script `openshift_deploy`.


Working with a mysql database
-----------------------------

Just issue:

    rhc cartridge add -a play2demo -c mysql-5.5

Don't forget to write down the credentials.

Then uncomment the following lines from your `conf/openshift.conf`:

    db.default.driver=com.mysql.jdbc.Driver
    db.default.url="jdbc:mysql://"${OPENSHIFT_DB_HOST}":"${OPENSHIFT_DB_PORT}/${OPENSHIFT_APP_NAME}
    db.default.user=${OPENSHIFT_DB_USERNAME}
    db.default.password=${OPENSHIFT_DB_PASSWORD}

You'll also have to include the mysql driver as a dependency. Add the folowing dependency in your `build.sbt` file:

    "mysql" % "mysql-connector-java" % "5.1.34" 

You can manage your new MySQL database by embedding `phpmyadmin-4`:

    rhc cartridge add -a play2demo -c phpmyadmin-4

It's also a good idea to create a different user with limited privileges on the database.


Updating your application
-------------------------

To deploy your changes to OpenShift just run the stage task, add your changes to the index, commit and push:

    activator clean stage
    git add . -A
    git commit -m "a nice message"
    git push origin

If you want to do a quick test, you can skip the `clean` and just run `activator stage`

All right, I know you are lazy, just like me. So I added a little script to help you with that, just run:

    openshift_deploy "a nice message"

You may leave the message empty and it will add something like "deployed on Thu Mar 29 04:07:30 ART 2012", you can also pass a `-q` parameter to skip the "clean" option.


A step by step example: deploying computer-database sample app to OpenShift
---------------------------------------------------------------------------

You can add OpenShift support to an already existing play application. 

Let's take the computer-database sample application.

```bash
    cd PLAY_INSTALL_FOLDER/samples/scala/computer-database

    git init
    rhc app create -a computerdb -t diy-0.1 --nogit
```

We add the "--nogit" parameter to tell OpenShift to create the remote repo but don't pull it locally. You'll see something like this:

```bash
    Confirming application 'computerdb' is available:  Success!

    computerdb published:  http://computerdb-yournamespace.rhcloud.com/
    git url:  ssh://uuid@computerdb-yournamespace.rhcloud.com/~/git/computerdb.git/
```
Copy and paste the git url to add it as a remote repo (replace the uuid part with your own!)

    git remote add origin ssh://uuid@computerdb-yourdomain.rhcloud.com/~/git/computerdb.git/
    git pull -s recursive -X theirs origin master
    git add .
    git commit -m "initial deploy"

That's it, you have just cloned your OpenShift repo, now we will add the quickstart repo:

    git remote add quickstart -m master git://github.com/opensas/play2-openshift-quickstart.git
    git pull -s recursive -X theirs quickstart master

Then run the stage task, add your changes to git's index, commit and push the repo upstream (you can also just run the *openshift_deploy* script):

    play clean compile stage
    git add .
    git commit -m "deploying computerdb application"
    git push origin

To see if the push was successful, open another console and check the logs with the following command:

    rhc app tail -a computerdb

Oops, looks like there's a problem.

```
[warn] play - Run with -DapplyEvolutions.default=true if you want to run them automatically (be careful)
Oops, cannot start the server.
PlayException: Database 'default' needs evolution! [An SQL script need to be run on your database.]
```

On development mode, play will ask you to run pending evolutions to database, but when in prod mode, you have to specify it form the command line. Let's configure play to automatically apply evolutions. Edit the file conf/openshift.conf like this:

```
# openshift action_hooks scripts configuration
# ~~~~~
openshift.play.params="-DapplyEvolutions.default=true"
```

Now deploy your app once again with './openshift_deploy -q'

That's it, you can now see computerdb demo application running at:

    http://computerdb-yournamespace.rhcloud.com

But there's one more thing you could do. Right now, your application is using the h2 in memory database that comes bundled with play. OpenShift may decide to swap your application out of memory if it detects there's no activity. So you'd better persist the information somewhere. That's easy, just edit your count/openshift.conf file to tell play to use a file database whenever it's running on OpenShift, like this:

```
db.default.driver=org.h2.Driver
db.default.url="jdbc:h2:"${OPENSHIFT_DATA_DIR}db/computerdb
```

Now, if you feel brave, you may port it to mysql. Add the mysql cartridge to you OpenShift application:

```
rhc app cartridge add -a computerdb -c mysql-5.1
```

There are a couple of differences you'll have to handle. Have a look at this quickstart to see what needs to be changed: https://github.com/opensas/openshift-play2-computerdb

I'll give you a few tips: the sample app uses H2 sequences instead of mysql auto_increment fields; you'll also have to modify the computer.insert method not to pass the id field; in order for the referential integrity to work you'll have to create the tables using the innodb engine; and you'll have to replace 'SET REFERENTIAL_INTEGRITY FALSE | TRUE' command with 'SET FOREIGN_KEY_CHECKS = 0 | 1;'.

Then edit you conf/openshift.conf file like this

    # openshift mysql database
    db.default.driver=com.mysql.jdbc.Driver
    db.default.url="jdbc:mysql://"${OPENSHIFT_DB_HOST}":"${OPENSHIFT_DB_PORT}/${OPENSHIFT_APP_NAME}
    db.default.user=${OPENSHIFT_DB_USERNAME}
    db.default.password=${OPENSHIFT_DB_PASSWORD}

You'll also have to include the mysql driver as a dependency. Add this line to project/Build.scala file:

    val appDependencies = Seq( 
        "mysql" % "mysql-connector-java" % "5.1.18" 
    ) 

You can manage your new MySQL database by embedding phpmyadmin-3.4.

    rhc app cartridge add -a computerdb -c phpmyadmin-3.4

Deploy once again, and you'll have your computerdb application running on OpenShift with mysql at:

    http://computerdb-yournamespace.rhcloud.com


Configuration
-------------

When running on OpenShift, the configuration defined with `conf/application.conf` will be overriden by `conf/openshift.conf`. This allows you to configure the way your play app will be executed while running on OpenShift.

You might want to pass extra arguments to start script that runs play application. To do this you can define `$PLAY_PARAMS` environment variable.

For example, to limit java memory usage to 512 MB you can do:

    rhc env set PLAY_PARAMS="-mem 512" -a play2demo


Trouble shooting
----------------

To find out what's going on in OpenShift, issue:

    rhc app tail -a play2demo

If you feel like investigating further, you can:

    rhc app show -a play2demo

```bash
play2demo @ http://play2demo-yourdomain.rhcloud.com/
  (uuid: youruuid)
---------------------------------------------------
  Domain:     yourdomain
  Created:    Nov 26 10:31 PM
  Gears:      1 (defaults to small)
  Git URL:    ssh://youruuid@play2demo-yourdomain.rhcloud.com/~/git/play2demo.git/
  SSH:        youruuid@play2demo-yourdomain.rhcloud.com
  Deployment: auto (on git push)

  diy-0.1 (Do-It-Yourself 0.1)
  ----------------------------
    Gears: 1 small
```

Then you can connect using ssh like this:

    ssh youruuid@play-yourdomain.rhcloud.com


Having a look under the hood
----------------------------

This projects takes advantage of OpenShift's do-it-yourself cartridge to run Play Framework 2 application natively.

Everytime you push changes to OpenShift, the following actions will take place:

* OpenShift will run the `.openshift/action_hooks/stop` script to stop the application, in case it's running.

* Then it will execute `.openshift/action_hooks/start` to start your application.

Play will then run your app in production mode.

`conf/openshift.conf` configuration will be used instead of `conf/application.conf`, `$PLAY_PARAMS` environment variable will define additional start script arguments.

The server will listen to `$OPENSHIFT_INTERNAL_PORT` at `$OPENSHIFT_INTERNAL_IP`.

`.openshift/action_hooks/stop` tries to kill the `RUNNING_PID` process, and then checks that the process is dead. If it's still alive, it tries forur more times to kill it nicely. If process still exists it tries another five times to kill it with `-SIGKILL`.


Acknowledgments
---------------

This quickstart is based on [Play Framework 2.0 quickstart](https://github.com/opensas/play2-openshift-quickstart) by opensas. Check it out to run Play 2.0.x and 2.1.x applications on OpenShift.


Licence
-------

This project is distributed under [Apache 2 licence](http://www.apache.org/licenses/LICENSE-2.0.html). 
