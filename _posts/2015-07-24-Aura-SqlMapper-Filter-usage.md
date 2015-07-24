---
layout: post
title: Simple Example of How to Use the Aura.SqlMapper_Bundle Filter
---

This documents a simple example of how to use the Aura\SqlMapper_Bundle\Filter
class to handle data changes to database tables where you want to take 
advantage of the database's ability to set a column to a default value.  In
this particular example, the database is Postgres and the column is a 
timestamp used to record the date and time of the change to the table row.

The Aura software is [here on Github](https://github.com/auraphp/Aura.SqlMapper_Bundle).


{% highlight php %}
<?php
use Aura\SqlMapper_Bundle\Filter;

/**
 * This is a Gateway-level filter.
 *
 * Removes columns which should not be sent to the database.  In this case,
 * the send_time column, which is defined as:
 *
 *      send_time       TIMESTAMP NOT NULL DEFAULT now()
 *
 * The database will set send_time to the current time upon insert or update,
 * if no value is provided.  This is what we want!
 *
 */
class MailLogGatewayFilter extends Filter
{
    /**
     * @param $subject - the table row entity as either an object or array.
     * @return mixed - the filtered $subject.
     */
    public function forInsert($subject)
    {
        return $this->globalFilter($subject);
    }

    /**
     * @param $subject - the table row entity as either an object or array.
     * @return mixed - the filtered $subject.
     */
    public function forUpdate($subject)
    {
        return $this->globalFilter($subject);
    }

    /**
     * Use the same filter for Insert or Update.
     *
     * @param $subject - the table row entity as either an object or array.
     * @return mixed - the filtered $subject.
     */
    private function globalFilter($subject)
    {
        if (is_array($subject)) {
            unset($subject['send_time']);
        }
        elseif (is_object($subject)) {
            unset($subject->send_time);
        }
        else {
            error_log(__FILE__ . ', ' . __FUNCTION__ . ': unexpected $subject type = ' . gettype($subject));
        }

        return $subject;
    }
}

//
// Using this filter in your Aura\SqlMapper_Bundle\AbstractGateway object.
//

$q = new ConnectedQueryFactory(new QueryFactory('sqlite'));

$f = new MailLogGatewayFilter();

$gateway = new MailLogGateway($this->connectionLocator, $q, $f);

?>
{% endhighlight %}
