---
layout: post
title: PHP PDO not always prepared
---

PHP's [PDO](http://php.net/manual/en/book.pdo.php) database API provides a consistent way for accessing databases, and has some nice features.  Among them is support for prepared statements.  According to the [documentation](http://php.net/manual/en/pdo.prepare.php), "PDO will emulate prepared statements/bound parameters for drivers that do not natively support them".  One might read this statement, and others in the documentation, to mean that if a database does support prepared statements, PDO will *not* emulate them.

Well, sort of.  The situation is complicated.

The first complication is for some database drivers, emulation is turned on by default, but can be explicitly turned off.  To turn off emulation mode, one must use the [PDO::setAttribute()](http://php.net/manual/en/pdo.setattribute.php) method.  Example:

{% highlight php %}
<?php
try {
    $pdo = new PDO("pgsql:host=server;dbname=mydatabase");
} catch (PDOExceoption $e) {
    echo("PDO Connection failed: " . $e->getMessage());
    exit(1);
}

if (! $pdo->setAttribute(PDO::ATTR_EMULATE_PREPARES, false)) {
    echo "Note: emulation mode not disabled.\n";
}
?>
{% endhighlight %}

Ok, so now PDO's emulation is disabled.  In theory, we can now use prepared statements, and take advantages of their benefits.  One benefit might to check SQL statements for correctness before actually trying to execute them.  In fact, database servers such as PostgreSQL and MySQL do this with prepared statements.

Let's try it with an intentional mistake to see what happens.

{% highlight php %}
<?php
$sth = $pdo->prepare('SELECT FROM :name');
if ($sth == false) {
    echo "prepare() failed\n";
    echo "[prepare] errorInfo = " . print_r($sth->errorInfo(), true);
}
?>
{% endhighlight %}

For our example code above using PostgreSQL, there is actually no output.  The call to prepare() does not throw an error.

We hit the second complication.

If this were MySQL, we would get an error message right there telling us the SQL syntax is wrong.  But with PostgreSQL, the PDO::prepare() call always succeeds.

The reason is because the PHP PDO driver for PostgreSQL *defers* the call to the server for prepared statements until during the [PDOStatement::execute()](http://php.net/manual/en/pdostatement.execute.php) call.  Here's the code from [php-src/ext/pdo_pgsql/pgsql_statement.c](https://github.com/php/php-src/blob/PHP-5.3.15/ext/pdo_pgsql/pgsql_statement.c):

{% highlight c %}
#if HAVE_PQPREPARE
    if (S->stmt_name) {
        /* using a prepared statement */

        if (!S->is_prepared) {
stmt_retry:
            /* we deferred the prepare until now, because we didn't
             * know anything about the parameter types; now we do */
            S->result = PQprepare(H->server, S->stmt_name, S->query,
                        stmt->bound_params ? zend_hash_num_elements(stmt->bound_params) : 0, S->param_types);
{% endhighlight %}

Well, that's a heckuva note, as they might say in northern Minnesota.  For PostgreSQL, PDO always *simulates* prepared statements, no matter what the setting of PDO::setAttribute(PDO::ATTR_EMULATE_PREPARES, *boolean*) is.  This makes it hard to write code with good error handling.

Instead, one has to check for all errors after the [PDOStatement::execute()](http://php.net/manual/en/pdostatement.execute.php) call.
