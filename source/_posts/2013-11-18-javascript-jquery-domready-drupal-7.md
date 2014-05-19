---
title: JavaScript, jQuery and DOM Ready in Drupal 7
allow_share: true
tags:
 - drupal
 - drupal-7
 - javascript
 - jquery
---

**tl;dr**: JavaScript code to process elements on page load on a Drupal 7 site should looks like this:
~~~.javascript
(function($) {
  Drupal.behaviors.doSomething = {
    attach: function(context, settings) {
      $('div.something', context).once('do-something').doSomething({
        param1: settings.somethingl.param,
        param2: 'something else'
      });
    }
  }
})(jQuery);
~~~

Drupal 7 provides jQuery in the [no-conflict mode](http://api.jquery.com/jQuery.noConflict/), which means that `$` is
not the jQuery object/namespace. This should not be an issue with properly written jQuery plugins that follow
[jQuery's plugins authoring documentation](http://learn.jquery.com/plugins/basic-plugin-creation/#protecting-the-alias-and-adding-scope).
This is however an issue for code snippets mindlessly copy/pasted from random web pages. Most of them expect `$` to be
the jQuery namespace and will not work within a Drupal page. This can be easily solved by wrapping theses snippets in
immediately invoked anonymous function that will alias the jQuery namespace to `$`:
~~~.javascript
(function($) {
    // Here $ is the jQuery namespace.
})(jQuery);
~~~

Usually, JavaScript code that needs to run at page load, is also wrapped in a function passed as argument to `jQuery()`
or `jQuery(document).ready()`:
~~~.javascript
$(function() {
  // Code here is executed when the DOM is loaded.
});
~~~

When combined, these two patterns are perfectly fine, even within Drupal. However if content (ie. new DOM elements) is
added to the page after page load (AJAX calls, content generated from JavaScript, etc.) the code in such functions will
never be able to process the added elements. Or if some portion of the content is removed or moved across the page, the
code will have no option to unregistered event handlers or update information about the already processed elements.
Drupal provides an API for this called [behaviors](https://drupal.org/node/304258#drupal-behaviors). Using behavior is
not required, but strongly recommended as a best practice to avoid future headaches (when code written six months ago
starts behaving strangely when a contrib module is added to the project). A behavior is written like this:
~~~.javascript
Drupal.behaviors.behaviorName = {
  attach: function (context, settings) {
    // Do something.
  },
  detach: function (context, settings, trigger) {
    // Undo something.
  }
};
~~~

The ``attach`` function of all registered behaviors (all properties of the ``Drupal.behaviors`` object) will be invoked
when behavior should be added to elements, either when the DOM is ready (ie. page load) and when elements are added to
the DOM. The ``detach`` function will be called when behaviors should be detached from elements: just before elements
are removed from the DOM, moved in the DOM or a form is submitted. The ``context`` parameter will always be a parent of
the added elements, the single added/removed/moved/submitted element itself or the whole ``document`` element. The
``settings`` parameters will be the settings for the ``context``, usually the ``Drupal.settings`` object as set by calls
to [``drupal_add_js()``](http://api.drupal.org/api/drupal/includes!common.inc/function/drupal_add_js/7) from PHP. For
``detach``, the ``trigger`` parameter will contains the kind of event that triggered the call: ``'unload'`` (elements
removed), ``'move'`` (elements moved) or ``'serialize'`` (form is being submitted).

The ``attach`` (and ``detach``) functions of a behavior can be used multiple time over the same portion of the DOM tree.
So the same element could be processed multiple time by the same code. It is up to the code itself to avoid processing
(ie. binding event handlers, altering CSS styles, etc.) multiple times for the same elements. The easiest solution for
this is to use the [jQuery Once](http://plugins.jquery.com/once/) plugin (which
[is provided by Drupal 7](http://drupal.org/node/756722#jquery-once)) like this:
~~~.javascript
$(selector).once('behavior-name').doSomething();
$(selector).once('behavior-name', function(){ /*do something*/ });
~~~

Since a behavior is being attached/detached to/from a context, the context object can be used to restrict your jQuery
queries to only the affected element or DOM subtree, like this:
~~~.javascript
$(selector, context).doSomething();
~~~

Putting all this together means the base pattern to process elements on page load should looks like this:
~~~.javascript
(function($) {
  Drupal.behaviors.doSomething = {
    attach: function(context, settings) {
      $('div.something', context).once('do-something').doSomething({
        param1: settings.somethingl.param,
        param2: 'something else'
      });
    }
  }
})(jQuery);
~~~
References:

- [Managing JavaScript in Drupal 7](http://drupal.org/node/756722)
- [Working with JavaScript](https://drupal.org/node/121997)
- [Converting 6.x modules to 7.x - Changed Drupal.behaviors to objects having the properties 'attach' and 'detach'](http://drupal.org/update/modules/6/7#drupal_behaviors)
- [http://api.drupal.org/api/drupal/includes!common.inc/function/drupal_add_js/7](http://api.drupal.org/api/drupal/includes!common.inc/function/drupal_add_js/7)
- [jQuery Once](http://plugins.jquery.com/once/)
