---
title: Behat step-definition to verify visibility (not just presence) of Drupal form elements.
allow_share: true
tags:
 - drupal
 - behat
 - drupal-7
---

In November 2013, I tried Drupal [BDD](https://en.wikipedia.org/wiki/Behavior-driven_development) using
[Behat](http://behat.org/) and the [Behat Drupal extension](https://drupal.org/project/drupalextension). The experiment
was to apply BDD in order to implement a small feature on an existing site. After a informal discussion about the
feature, the stakeholder wanting it wrote the Cucumber scenarios. As the developer of the feature, I took the
scenarios and started by running them against my test site. And so, my BDD journey started.

The first thing that needed fixing was the scenario itself, off course being the first scenario ever written by its
author, most of the step, while understandable by an human being did not match the pattern expected by Behat (actually
the (Mink)[http://mink.behat.org/] and Drupal extensions). While fixing the steps, I quickly learned that steps to easily
express the visibility of Drupal form element did not exists. So I wrote a couple or steps to check for the
(in)visibility of form element identified by their labels. Writing the steps was the occasion to dig the Behat and Mink
APIs. Most of them requires a (Mink driver)[http://mink.behat.org/#different-browsers-drivers] with support for CSS in
order for the visibility tests on the form element to work. The Goutte driver does not, the Selenium driver does not. I
don't known for others.

The code for these steps is distributed under a MIT-style licence as Gist (embed below). In
(issue 2151935)[https://drupal.org/node/2151935] Another Behat/Drupal enthusiast requested for them to be included . This
would require a bit more work as they currently heavily rely on, what I think is, the default Drupal form markup and have
only been tested in single custom theme. Because we can expect everyone to use default form markup, the code to retrieve
a form element or its label should abstracted to easily overwriteable methods. One of these methods should be used to
retrieve the form labels on the page. Once you have a form label element, you can retrieve its form element using its
`for` attribute (requiring this attribute is a reasonable requirements). Another method should be used to retrieve the
type (ie. the Drupal form API `#type`) of a form element.

<script src="https://gist.github.com/pbuyle/7698675.js"></script>


heavily relies on the default Drupal markup for form elements.