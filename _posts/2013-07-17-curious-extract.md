---
layout: post
title: The Curious Case of PHP's extract()
---

I'm working on legacy PHP code used for an extranet website.  A previous developer's code handles many forms by ultimately globalizing the form variables in a manner similar to the code sketch below:

{% highlight php %}
<?php
$params = null;
$count = extract($_REQUEST, EXTR_IF_EXISTS);

// ...

if (is_array($params)) {
    extract($params);
}
?>
{% endhighlight %}

I believe this rather gratuitous use of extract() leaves something to be desired.  If it's used _very_ carefully, it can be made secure from various injection attacks.  But it's easy to make a mistake.  So I thought I'd see if I could find another way to make legacy code still work, but avoid the weak points.

Many of this application's forms are fairly complex, so I also wondered if the performance of a handful or more lines of PHP code would hurt performance, compared to the compiled-in extract() command.  This lead to a quick benchmark to get a rough idea of what I was facing.

Strangely, the PHP coded "replacement" for extract() was faster than the actual command.  I have some theories about why this is true, but fortunately in most cases the extra work that extract() is doing is not really needed, so PHP code will meet all needs, be safer and possibly even faster.

The time difference is small for 10,000 iterations, so will be very small for the one or two calls that the typical legacy web script was using.

Here's my quick and dirty benchmark.

{% highlight php %}
<?php
$REQ = array(
    'tid' => 1372460988,
    'dest' => 'report',
    'source' => 'bill',
    'params' => array(
                    'client' => '897462694',
                    'client_ref_id' => array(
                                        0 => '3707837'
                                    ),
                    'invoice_ref_id' => '',
                    'user_id_0' => 'A5026',
                    'user_id_1' => 'K4265',
                    'user_id_2' => '',
                    'user_id_3' => '',
                    'user_id_4' => '',
                    'user_id_5' => '',
                    'user_id_6' => '',
                    'user_id_7' => '',
                    'user_id_8' => '',
                    'user_id_9' => '',
                    'user' => '',
                    'indicator' => 'CI',
                    'status' => 'ACCEPT',
                    'invoice_npi' => '',
                    'biller_npi' => '',
                    'column' => 'date_registered',
                    'from_date' => '2013-06-01',
                    'to_date'   => '2013-06-28',
                    'group_by'  => 'user_id_group'
                ),
    'submit'    => 'bill_category_report',
    'src'       => 'session',
    'SERVER_COOKIE'  => 'ohq7j7ai23qoc1tkjl499ld8s3'
);

function test1($iterations, $var) {
    for ($i=0; $i<$iterations; $i++) {
        extract($var);
    }
}

function test2($iterations, $var) {
    for ($i=0; $i<$iterations; $i++) {
        foreach ($var as $key => $val) {
            $$key = $val;
        }
    }
}

$iterations = 50000;

$start = microtime(true);
test1($iterations, $REQ['params']);
$delta = microtime(true) - $start;

echo "test1 (extract) elapsed time: " . sprintf("%8.5f\n", $delta);

$start = microtime(true);
test2($iterations, $REQ['params']);
$delta = microtime(true) - $start;

echo "test2 (loop) elapsed time:    " . sprintf("%8.5f\n", $delta);
?>
{% endhighlight %}

Sample output:

    php extract.php
    test1 (extract) elapsed time:  0.17561
    test2 (loop) elapsed time:     0.09619
