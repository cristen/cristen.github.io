---
layout: post
title:  "How to migrate your PostgreSQL database along with its extensions"
date: 2017-12-04 06:48:00 GMT+1
categories: PostgreSQL
---

## I broke the system captain

So here you are, facing the awfull truth about your postgresql install.

Yes my friend you intentionnally \**wink*\* forgot \**wink*\* to add the IgnorePkg directive as stated in the official [Archwiki](https://wiki.archlinux.org/index.php/PostgreSQL#Upgrading_PostgreSQL) :

{% highlight bash %}
IgnorePkg = postgresql postgresql-libs
{% endhighlight %}

And on this day, you just did your usual upgrade routine :

{% highlight shell %}
pacman -Syu
{% endhighlight %}

Same routine with your database upgrade, you follow the tutorial line by line and stumble on this command :

{% highlight shell %}
pg_upgrade -b /opt/pgsql-9.6/bin -B /usr/bin -d /var/lib/postgres/olddata -D /var/lib/postgres/data
{% endhighlight %}

## What should I do first ?

Just take a look at the "pg_upgrade_internal.log" file that should reside in the current directory (the one that you used to launch the previous command).  
By any chance if the problem lies with a library the fix is pretty straightforward.

## Possible fix if it’s a library ?

There are three things that you should know about libraries or extensions :

+ They must be installed on both the **new** and the **old** PostgreSQL clusters.
+ It includes doing the "CREATE EXTENSION <libname>" command on both databases (refer to the extension documentation about this).
+ You must use the binaries specific to each version for the install.

To fire up the old cluster, if you installed the **postgresql-old-upgrade** package, you’ll find the old binaries here :

{% highlight shell %}
/opt/pgsql-<version>/bin/
{% endhighlight %}

For instance on PostgreSQL 9.6 you can fire up your old cluster instance like this :

{% highlight shell %}
# Fire up your old cluster, ⚠ you should first stop your current database instance
# by doing so : systemctl stop postgresql.service
/opt/pgsql-9.6/bin/pg_ctl -D <path_to_your_data_folder>
# Then you can use the psql from the old path
/opt/pgsql-9.6/bin/psql
{% endhighlight %}

And that’s all, once your extensions are all set on both your clusters it should continue without problem.

By the way a quicker way would probably be to dump your databases first, and then restore these on your new cluster if those databases are fairly light.

## Last words

As always if you don’t know what you’re doing, you should probably seek advice from database admins. Especially, don’t try this on a production server. I’m a developer, just sharing my knowledge on this subject and I will not be held responsible for any of your actions. Those are only suggestions to where you should look at if this gets into your way.
