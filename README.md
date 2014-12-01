Play Framework 2 application on OpenShift Express
=================================================

**WARNING: this quickstrt is in the middle of updating and not yet ready to be used. The plan is to complete it in at the beginning of December 2014.**

This git repository will help you get up and running quickly with a Play Framework 2 application
on OpenShift Express taking advantage of the do-it-yourself cartridge.

TODO: add info about supported versions/where to go for 2.0.x/2.1.x

TODO: add info about tested versions

TODO: add info about `play` command deprecation

- [Running on OpenShift](#running-on-openshift)
- [Working with a mysql database](#working-with-a-mysql-database)
- [Updating your application](#updating-your-application)
- [Configuration](#configuration)
- [Troubleshooting](#troubleshooting)
- [A step by step example: deploying "play-scala-intro" sample app to OpenShift](#a-step-by-step-example-deploying-play-scala-intro-sample-app-to-openshift)
- [Having a look under the hood](#having-a-look-under-the-hood)
- [Acknowledgments](#acknowledgments)
- [Licence](#licence)


Running on OpenShift
--------------------

Create a new Play Framework 2 application:

    activator new play2demo
    cd play2demo

    git init
    git add .
    git commit -m "project import"

Register at https://openshift.redhat.com/, and then create a diy (do-it-yourself) application:

    rhc app create play2demo -t diy-0.1 --no-git -l yourlogin

You will see something like the following:

```
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

Copy and paste the "Git remote" url to add it as a remote repo (replace the `uuid` part with your own!):

    git remote add origin ssh://your_uuid@play2demo-yourdomain.rhcloud.com/~/git/play2demo.git/
    git pull -s recursive -X theirs origin master

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

```
db.default.driver=com.mysql.jdbc.Driver
db.default.url="jdbc:mysql://"${OPENSHIFT_MYSQL_DB_HOST}":"${OPENSHIFT_MYSQL_DB_PORT}/${OPENSHIFT_APP_NAME}
db.default.user=${OPENSHIFT_MYSQL_DB_USERNAME}
db.default.password=${OPENSHIFT_MYSQL_DB_PASSWORD}
```

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

If you want to do a quick test, you can skip the `clean` and just run `activator stage`.

All right, I know you are lazy, just like me. So I added a little script to help you with that, just run:

    openshift_deploy "a nice message"

You may leave the message empty and it will add something like "deployed on Thu Mar 29 04:07:30 ART 2012", you can also pass a `-q` parameter to skip the `clean` option.


Configuration
-------------

When running on OpenShift, the configuration defined with `conf/application.conf` will be overriden by `conf/openshift.conf`. This allows you to configure the way your Play app will be executed while running on OpenShift.

You might want to pass extra arguments to start script that runs Play application. To do this you can define `$PLAY_PARAMS` environment variable.

For example, to limit java memory usage to 512 MB you can do:

    rhc env set PLAY_PARAMS="-mem 512" -a play2demo


Troubleshooting
---------------

To find out what's going on in OpenShift, issue:

    rhc tail play2demo

If you feel like investigating further, you can:

    rhc app show -a play2demo

```
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


A step by step example: deploying "play-scala-intro" sample app to OpenShift
----------------------------------------------------------------------------

You can add OpenShift support to an already existing Play application.

Let's create `intro` sample application based on `play-scala-intro` template:

    activator new intro play-scala-intro
    cd intro

Then, create git repo and add project:

    git init
    git add .
    git commit -m "project import"
    
Now, lets create application on OpenShift:

    rhc app create -a intro -t diy-0.1 --no-git

We add the `--nogit` parameter to tell OpenShift to create the remote repo but don't pull it locally. You'll see something like this:

```
Application Options
-------------------
Domain:     yourdomain
Cartridges: diy-0.1
Gear Size:  default
Scaling:    no

Creating application 'intro' ... done

  Disclaimer: This is an experimental cartridge that provides a way to try unsupported languages, frameworks, and middleware on OpenShift.

Waiting for your DNS name to be available ... done

Your application 'intro' is now available.

  URL:        http://intro-yourdomain.rhcloud.com/
  SSH to:     youruuid@intro-yourdomain.rhcloud.com
  Git remote: ssh://youruuid@intro-yourdomain.rhcloud.com/~/git/intro.git/

Run 'rhc show-app intro' for more details about your app.
```

Copy and paste the "Git remote" url to add it as a remote repo (replace the uuid part with your own!):

    git remote add origin ssh://youruuid@intro-yourdomain.rhcloud.com/~/git/intro.git/
    git pull -s recursive -X theirs origin master

That's it, you've just cloned your OpenShift repo, now we will add the quickstart repo:

    git remote add quickstart -m master git://github.com/EugenyLoy/play2-openshift-quickstart.git
    git pull -s recursive -X theirs quickstart master

Then run the `stage` task, add your changes to git's index, commit and push the repo upstream (you can also just run the `openshift_deploy` script):

    activator clean stage
    git add .
    git commit -m "deploying intro application"
    git push origin

To see if the push was successful, open another console and check the logs with the following command:

    rhc tail intro

Everything should be ok. You can now see `intro` application running at:

    http://intro-yourdomain.rhcloud.com


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

This quickstart is based on [Play Framework 2.0 quickstart](https://github.com/opensas/play2-openshift-quickstart) by opensas. Check it out to run applications based on older (pre-2.2.x) Play Framework versions on OpenShift.


Licence
-------

This project is distributed under [Apache 2 licence](http://www.apache.org/licenses/LICENSE-2.0.html). 
