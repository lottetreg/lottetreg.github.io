---
layout: post
title:  "Moving Data Between Remote Databases with Your Local File System"
date:   2017-08-08
categories: postgres
---

I needed to migrate data from one of our production Postgres databases to another. My initial thought was to do all of this from the Rails console of each application by storing and accessing query results on Amazon S3. Instead, I decided to transfer the data through my local file system.

You may be familiar with the `COPY` SQL command, which allows you to copy data to and from files (in this case, I wanted it to be in a CSV format). It works like this:

{% highlight sql %}
COPY (SELECT * FROM dogs)
TO '/absolute/path/dogs.csv'
WITH (FORMAT csv, HEADER true);
{% endhighlight %}

This approach writes to a file that is local to the database, which in this case would have meant storing it on a remote server. I wanted to store the data directly on my computer.

PostgreSQL provides us with `\copy`, a handy meta-command that lets you use your local file system with remote database connections. It works like this:

{% highlight postgresql %}
\copy (SELECT * FROM dogs) TO dogs.csv WITH (FORMAT csv, HEADER true)
{% endhighlight %}

Notice that the entire command is now on one line——these psql backslash commands are triggered with a new line rather than a semicolon. We also get to forgoe the quotes, and we can now use the relative file path instead of the absolute one.

I opted to use the `\copy` command along with the PostgreSQL intereactive terminal (denoted by the `psql`). This single command connects you to the database _and_ copies the query result to a local file called `/tmp/dog_data.csv`:

{% highlight bash %}
  psql `heroku config:get DATABASE_URL -a APPLICATION_1` -c "\copy (
    select name, birthday from dogs
  ) to /tmp/dog_data.csv with (format csv, header true)"
{% endhighlight %}

The `-c` option tells psql to execute the following command string. You can read more about it [here](https://www.postgresql.org/docs/9.6/static/app-psql.html). Also, notice that we no longer have to keep the entire command on one line; however you should still omit the semicolon at the end.

I then used the same `\copy` command to copy data _from_ the CSV _to_ my other remote database:

{% highlight bash %}
  psql `heroku config:get DATABASE_URL -a APPLICATION_2` -c "\copy tmp_dogs (
    name, birthday
  ) from /tmp/dog_data.csv with (format csv, header true)"
{% endhighlight %}

Because I needed to run some more queries on this data before inserting the new records, I created a new table (tmp_dogs) and copied the data into there. Once the new records had been created, I deleted the tmp_dogs table, and my task is complete!
