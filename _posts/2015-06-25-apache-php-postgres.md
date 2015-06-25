---
layout: post
title: The Curious Case of Apache, PHP and Postgres
---

Like many development shops, we use the Apache PHP module (modphp) with
our webserver for our PHP-driven web applications.  We also use Postgres as
our primary database.  We have a mix of other languages which also use
the Postgres database, either as server applications or batch processes.

We wanted to better secure our database, and have fewer passwords floating
around in files and active applications.

The Postgres API library (written in C, `libpq`) provides several methods of
doing authentication which are helpful in improving security.  We evaluated
two of these.

One method was the
[Password File (.pgpass)](http://www.postgresql.org/docs/9.4/static/libpq-pgpass.html)
and the other was use of the
[Connection Service File (pg_service.conf)](http://www.postgresql.org/docs/9.4/static/libpq-pgservice.html)
.

Both methods are portable across any language which uses the libpq library, so
it seemed like one or the other, or both, would be a good fit for our
environment.  In initial testing from the command line, all appeared to work
well.  Then we started testing Apache PHP, and encountered strange problems.

First, PHP could not find the Password File or the Connection Service File.
The Postgres library does automatic file path searching based on environment
variables.  Calling `getenv()` from PHP displayed the correct values.

After some searching of the web, I found that when using the Apache PHP module,
to actually get the values down to the Postgres library, I had to explicitly
`putenv()` the values.  I ended up with the following odd code which first
obtained the correct values from the
enviroment and then just put them right back.  

For the Password File, my code looked like this:

```php
    putenv('PGPASSFILE=' . getenv('PGPASSFILE'));
```

And for the Connection Service File, it looked like this:

```php
    putenv('PGSYSCONFDIR=' . getenv('PGSYSCONFDIR'));
```

My tests with the above code for the Password File seemed to work.  Then a
co-worker modified his web application to use the same technique, and it
failed.  After trying various things, we discovered it would sometimes
work and sometimes fail, and it would do so at different points in the
code.  It also seemed sensitive to where in the code the `putenv()` was
called, i.e. at the top of the script or just before needing to make a
database connection.

Suffice to say, this left us scratching our heads with no resolution and
a healthy distrust for using the Password File.

Initial tests in web applications with the Connection Service File looked
slightly more promising, but due to the strange behavior we saw with the
Password File, and that both methods required the `putenv()`, our
confidence was low.  We have so far not done further extensive testing of
the Connection Service File method.  Insetead we decided for the moment to
live with the duplicated efforts of maintaining two password files, one
for the rest of the operation that can use either the Password File
(.pgpass) or the Connection Service File (pg_service.conf) successfully --
that is, applications that are not running PHP from Apache.  The second
copy is a INI format file read by a custom configuration class.

This result leaves me unsatisfied, so I have no given up on getting to
the bottom of this problem and finding a better solution.
