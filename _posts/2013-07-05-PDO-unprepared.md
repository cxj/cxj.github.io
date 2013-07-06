---
layout: post
title: PHP PDO not always prepared
---

PHP's [PDO](http://php.net/manual/en/book.pdo.php) database API provides a lot of nice features, or so it appears on the surface.  According to the [documentation](http://php.net/manual/en/pdo.prepare.php), "PDO will emulate prepared statements/bound parameters for drivers that do not natively support them".  One might read this statement, and others in the documentation, to mean that if a database supports prepared statements, PDO will not emulate them.

Well, sort of.

It turns out the situation is quite a bit more complicated than that.

The first complication is that for some database drivers, emulation is turned on by default, but can be explicitly turned off.  To turn off emulation mode, one must use the [PDO::setAttribute()](http://php.net/manual/en/pdo.setattribute.php) method.  Example:

{% highlight php %}
try {
    $pdo = new PDO("pgsql:host=server;dbname=mydatabase");
} catch (PDOExceoption $e) {
    echo("PDO Connection failed: " . $e->getMessage());
    exit(1);
}

if (! $pdo->setAttribute(PDO::ATTR_EMULATE_PREPARES, false)) {
    echo "Note: emulation mode not disabled.\n";
}
{% endhighlight %}

Ok, so now we are no longer using emulation.  In theory, we can now use prepared statements, and take advantages of their benefits.  One benefit might to check SQL statements for correctness before actually trying to execute them.  In fact, prepared statements on database servers such as PostgreSQL and MySQL do this for you.

Let's try it:

{% highlight php %}
$sth = $pdo->prepare('SELECT FROM :name');
if ($sth == false) {
    echo "prepare() failed\n";
    echo "[prepare] errorInfo = " . print_r($sth->errorInfo(), true);
}
{% endhighlight %}

Now we hit the second complication.  If this were MySQL, we would get an error message right here telling us the SQL syntax is wrong.  But with PostgreSQL, the PDO::prepare() call always succeeds.  The reason is because the PHP PDO driver for PostgreSQL *defers* the call to the server for prepared statements until during the PDO execute call, right before PDO calls the PostgreSQL library's execute.  Here's the code from hp-src/ext/pdo_pgsql/pgsql_statement.c:

{% highlight c %}
#if HAVE_PQPREPARE
    if (S->stmt_name) {
        /* using a prepared statement */

        if (!S->is_prepared) {
stmt_retry:
            /* we deferred the prepare until now, because we didn't
             * know anything about the parameter types; now we do */
            S->result = PQprepare(H->server, S->stmt_name, S->query,
                        stmt->bound_params ? zend_hash_num_elements(stmt-       >bound_params) : 0,
                        S->param_types);
{% endhighlight %}

Well, that's a heckuva note, as they might say in northern Minnesota.  For PostgreSQL, PDO always *simulates* prepared statements, no matter what the setting of PDO::setAttribute(PDO::ATTR_EMULATE_PREPARES, *boolean*) is.  This makes it hard to write code with good error handling.

Instead, one has to check for all errors after the [PDOStatement::execute()](http://php.net/manual/en/pdostatement.execute.php) call.  I find this less than optimal, especially since PostgreSQL has supported prepared statemetns longer than MySQL.  It might be simply a difference in opinion by different programmers who wrote the MySQL and PostgreSQL PDO drivers.  There's no technical reason why PostgreSQL couldn't provide the same PDO support as MySQL.
