---
layout: post
title: Programmatic CCK content type creation
---

Do you want your Drupal module to create a new content type using the power and flexibility of CCK, perhaps including the built-in integration with Views? Here's one way, using Drupal 6.

----
## Step 1: Design your CCK content type using the Drupal content type editor

The menu path is: Administer -> Content management -> Content types -> Add content type, URL http://example.com/admin/content/types/add. For more information about getting started with building a content type with CCK, visit [this Drupal handbook page](http://drupal.org/node/162242).

----
## Step 2: Export your new CCK content type

After you save your new content type in step 1 above, then navigate to menu path Administer -> Content management -> Content types -> Export.  Your URL  should look like this:  http://example.com/admin/content/types/export

Select the content type you just created from the radio buttons displayed, and then click the Export button.

The next screen displays the fields for the selected content type.

Take note of the Types of fields you used. If there are any which are not included with the basic CCK module (content.module), you need to make sure to include them in the dependencies for your module below, or it will be unable to create the content type.

In general, you will want to use the default of exporting all of the fields. Make sure they are all selected via the checkboxes, then click the Export button.

The next page produces a large textarea with the necessary PHP code to define your CCK content type. When ready to use this code, select all of the text in the textarea and copy it to your copy buffer to paste into your definition function, described below.

----
## Step 3: create a PHP function to hold your CCK definition

The template of the function will look like this, prior to pasting the contents of your copy buffer from above (containing the exported CCK content type definition code). We use a separate function just for the definition because it is must be updated with a new export from CCK if we decide to change or tweak our CCK content type later. It makes editing easier and less error prone.

{% highlight php %}
<?php
    function _modulename_cck_export() {
      // paste code after this line.

      // paste code before this line.
      return $content;
    }
?>
{% endhighlight %}

----
## Step 4: Carefully paste the exported CCK content type code into the function

The end result should look like this:

{% highlight php %}
<?php
    function _modulename_cck_export() {
      // paste code after this line.
      $content[type]  = array (
        'name' => 'computed moodle',
        'type' => 'computemoodle',
        'description' => 'Example CCK code.',
        'title_label' => 'Title',
        'body_label' => 'Body',
        'min_word_count' => '0',
        'help' => '',
        'node_options' => 
        array (
          'status' => true,
          'promote' => true,
          'sticky' => false,
          'revision' => false,
        ),
        'old_type' => '',
        'orig_type' => '',
        'module' => 'node',
        'custom' => '1',
        'modified' => '1',
        'locked' => '0',
        'content_profile' => false,
        'comment' => '2',
        'comment_default_mode' => '4',
        'comment_default_order' => '1',
        'comment_default_per_page' => '50',
        'comment_controls' => '3',
        'comment_anonymous' => 0,
        'comment_subject_field' => '1',
        'comment_preview' => '1',
        'comment_form_location' => '0',
      );
      $content[fields]  = array (
        0 => 
        array (
          'label' => 'Example field',
          'field_name' => 'field_example',
          'type' => 'number_integer',
          'widget_type' => 'number',
          'change' => 'Change basic information',
          'weight' => '-1',
          'description' => '',
          'default_value' => 
          array (
            0 => 
            array (
              'value' => '',
              '_error_element' => 'default_value_widget][field_example][0][value',
            ),
          ),
          'default_value_php' => '',
          'default_value_widget' => NULL,
          'group' => false,
          'required' => 0,
          'multiple' => '0',
          'min' => '',
          'max' => '',
          'prefix' => '',
          'suffix' => '',
          'allowed_values' => '',
          'allowed_values_php' => '',
          'op' => 'Save field settings',
          'module' => 'number',
          'widget_module' => 'number',
          'columns' => 
          array (
            'value' => 
            array (
              'type' => 'int',
              'not null' => false,
              'sortable' => true,
            ),
          ),
          'display_settings' => 
          array (
            'label' => 
            array (
              'format' => 'above',
              'exclude' => 0,
            ),
            'teaser' => 
            array (
              'format' => 'default',
              'exclude' => 0,
            ),
            'full' => 
            array (
              'format' => 'default',
              'exclude' => 0,
            ),
            4 => 
            array (
              'format' => 'default',
              'exclude' => 0,
            ),
          ),
        ),
      );
      $content[extra]  = array (
        'title' => '-5',
        'body_field' => '-3',
        'menu' => '-2',
      );
      // paste code before this line.
      return $content;
    }
?>
{% endhighlight %}

----
## Step 5: Optionally clean up the pasted code

You may want to edit any string array indexes to be enclosed in single quote marks to avoid PHP warnings. I'm not sure why CCK exports faulty code like this. For example, as exported, the last array element is given as

{% highlight php %}
<?php
    $content[extra] = array (
?>
{% endhighlight %}

but really should be expressed as

{% highlight php %}
<?php
    $content['extra'] = array (
?>
{% endhighlight %}


----
## Step 6: Write PHP code to actually create the content type

Write the install code to initiate creation of the new content type. This is where we make sure we have the necessary CCK pieces, populate a form using the now hard-coded CCK export, and then use drupal_execute() to submit the form to create the new content type. Here's an example:

{% highlight php %}
<?php
    function _example_install_cck_node() {
      /* get the CCK node types to be created.  This is where you load the 
       * file containing your function from above, if necessary, and then call
       * that function.
       */
      module_load_include('inc', 'modulename', 'modulename.ccknodedef');
      $content = _modulename_cck_export();  // in modulename.ccknodedef.inc

      // CCK content_copy.module may not be enabled, so make sure it is included
      require_once './' . drupal_get_path('module', 'content') 
          . '/modules/content_copy/content_copy.module';

      $form_state['values']['type_name'] = '<create>';
      $form_state['values']['macro'] = '$content = '
                                      . var_export($content, TRUE)
                                      . ';';

      // form provided by content_copy.module  
      drupal_execute('content_copy_import_form', $form_state);
      content_clear_type_cache();
    }
?>
{% endhighlight %}

----
## Step 7: Call the create function from your module

Figure out where you want to call the above install function from. I created an administrator button for creating it, because hook_install() uses the Batch API which is incompatible with the Forms API used to create the content type. Thus, it's not possible to have the type easily created at install time when the module is enabled. See the bug at issue [297972](http://drupal.org/node/297972). This bug is actually now fixed in Drupal CVS, but is not yet in a release as of Drupal 6.10.

----
## Step 8: Edit your *.info file

Be sure to add the appropriate dependencies[] clauses in your modulename.info file if your CCK content type is not optional for your module. At a minimum, you will need to specify the CCK module, which is named content.

----
## All done.

That's it. Now your module can create a CCK content type it can use, and which will benefit from all of the other functionality such as Views which works so well with CCK.

Let me know if you have questions, corrections or improvements.

Thanks to [Angie Byron (webchick)](http://drupal.org/user/24967) for pointing me at some existing code which helped me finish figuring out how to do this cleanly.
