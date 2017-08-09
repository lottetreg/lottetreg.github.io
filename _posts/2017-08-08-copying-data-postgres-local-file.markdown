---
layout: post
title:  "Copying Data in Postgres with Your Local File System"
date:   2017-08-08
categories: postgres
---

As I've moved further down the stack in my career, I've had the opportunity to tackle more and more interesting problems—many of which I've solved with PostgreSQL. Today's challenge was not an exception!

I needed to figure out a way to migrate data from one of our Postgres production databases to another. My initial thought was to do all of this from the Rails console, storing and accessing query results on Amazon S3. Luckily I had a second thought, one that would allow me to transfer data bewteen remote databases all through my local file system.

You may be familiar with the `COPY` SQL command, which allows you to copy data to and from files (in this case, I wanted to use a CSV format). It works like this:

{% highlight sql %}
COPY (SELECT * FROM dogs)
TO '/absolute/path/dogs.csv'
WITH (FORMAT csv, HEADER true);
{% endhighlight %}

Unfortunately this approach has a major drawback; it will only write to a file that is local to the database. That means that if we wanted to copy data from a remote database, we could only store that data in a remote file. It would be much simpler to store the data in a file on my computer.

PostgreSQL provides us with a handy meta-command that lets us use our local file system with remote database connections: `\copy`. It works like this:

{% highlight postgresql %}
\copy (SELECT * FROM dogs) TO dogs.csv WITH (FORMAT csv, HEADER true)
{% endhighlight %}

Notice that the entire command is now on one line——these psql backslash commands are triggered on a new line rather than a semicolon. We also get to forgoe the quotes, and we can now use the relative file path instead of the absolute one.

To make my task even simpler, I opted to use the `\copy` command along with the PostgreSQL intereactive terminal (denoted by the `psql`). This allows you achieve the same results from the command line:

{% highlight bash %}
  psql `heroku config:get DATABASE_URL -a barkbox` -c "\copy (
    select dog_name, dog_birthday, user_id from subscriptions
  ) to /tmp/dog_data.csv with (format csv, header true)"
{% endhighlight %}

The `-c` option tells psql to execute the following command string. You can read more about it [here](https://www.postgresql.org/docs/9.6/static/app-psql.html). Also, note that I no longer had to keep the entire command on one line; however you should still omit the semicolon at the end.

And now I have a CSV in my /tmp directory with the results of my query!

Now I can use the same `\copy` command to copy data _from_ the CSV _to_ my other remote database:

{% highlight bash %}
  psql `heroku config:get DATABASE_URL -a barkshop` -c "\copy tmp_dogs (
    dog_name, dog_birthday, user_id
  ) from /tmp/dog_data.csv with (format csv, header true)"
{% endhighlight %}

Because I needed to run some more queries on this data before inserting the new records, I created a new table (tmp_dogs) and copied the data into there.

Finally, once the new records have been created, I delete the tmp_dogs table, and my task is complete!
