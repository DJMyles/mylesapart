---
title: "Drupal 8 - Taming user forms for theming"
date: 2019-01-01
tags: ["cms","drupal","front end","php","html5","twig"]
draft: false
---

![](/img/drupal_8_logo_stacked.png#articleimg)

I've spent my holidays working on a project I've been thinking about and planning for quite a while now. Whilst I'm not quite ready to reveal the specifics of said project quite yet (more on that early 2019), I'm keen to share my experiences on how to wrangle Drupal 8's many user forms in situations where you want to present a particular front facing theme to end users who will be making their own content using your CMS.

I'm used to front end development in an environment where the end user is purely a consumer of the content. Any authentication, publishing and content creation is completely reserved for internal and is completely decoupled from the front end. With my newest project I'm creating a framework where the end user becomes an authenticated user and creates their own content.

Theming the user forms isn't as significant an issue when your publishers or content creators are internal. You can simply switch on the Seven or Adminimal theme for these users and you're good to go out of the box with minimal alterations needed.

> But what if you want to use your front facing theme for authentication and content creation so that your authenticated end users get a consistent experience?

You will need to incorporate your theme into many of Drupal 8 forms

What makes theming the user forms tricky is that depending on the form in question - the approach changes.

[Kint](https://kint-php.github.io/kint/) is your friend but even then, navigating its maze of variables, objects, strings and arrays, not to mention protected areas (more on that below), can be tricky for dabblers like me who hate writing PHP and want to spend more time writing awesome looking SASS. [Documentation and discussion](https://drupal.stackexchange.com) around best practice seems varied and fragmented, with a lot of techniques specific to D7 or D8.

So what forms are we talking about?

A vanilla Drupal 8 install generally has 4 main user forms.

* `user_login_form`
* `user_pass`
* `user_register_form`
* `user_form`

Depending on what flavour of Drupal 8 you have installed and what module stack you have running, the respective forms will look something like this:

![](/img/login.png#bodyimg)
![](/img/password.jpg#bodyimg)
![](/img/account.png#bodyimg)
![](/img/user.png#bodyimg)

So lets explore how to change up these forms so that they look a little less crappy.

The bulk of our code will be making use of the [hook_form_alter function](https://api.drupal.org/api/drupal/core%21lib%21Drupal%21Core%21Form%21form.api.php/function/hook_form_alter/8.6.x) made available to us via the form API.

```php
/**
 * Implements hook_form_alter().
 */
function mymodule_form_alter(&$form, $form_state, $form_id) {
  // Your code here
}
```

The problem with just whacking changes into this function without any conditionals is that we will inadvertently change a bunch of elements of other forms we don't want to touch such as node edit forms for example. We can fix this pretty easily by using a case statement for each user form:

```php
/**
 * Implements hook_form_alter().
 */
function mymodule_form_alter(&$form, $form_state, $form_id) {
  switch($form_id) {
    case 'user_login_form':
      // Your code here
    break;
    case 'user_pass':
      // Your code here
    break;
    case 'user_register_form':
      // Your code here
    break;
    case 'user_form':
      // Your code here
    break;
  }
}
```
Lets add some placeholder text to the input elements in `the user_login_form` with the name attributes of `name` and `pass` (easily found via your browser inspector). Our code would be:

```php
/**
 * Implements hook_form_alter().
 */
function mymodule_form_alter(&$form, $form_state, $form_id) {
  switch($form_id) {
    case 'user_login_form':
      $form['name']['#attributes']['placeholder'] = t('Foo');
      $form['pass']['#attributes']['placeholder'] = t('Bar');
    break;
    case 'user_pass':
      // Your code here
    break;
    case 'user_register_form':
      // Your code here
    break;
    case 'user_form':
      // Your code here
    break;
  }
}
```

Now that we have some Placeholder text within the input we might not want our input labels visible anymore. But don't we need them for accessibility? No problem! Consider the following code:

```php
/**
 * Implements hook_form_alter().
 */
function mymodule_form_alter(&$form, $form_state, $form_id) {
  switch($form_id) {
    case 'user_login_form':
      $form['name']['#attributes']['placeholder'] = t('Foo');
      $form['pass']['#attributes']['placeholder'] = t('Bar');
      $form['name']['#title_display'] = 'invisible';
      $form['pass']['#title_display'] = 'invisible';
    break;
    case 'user_pass':
      // Your code here
    break;
    case 'user_register_form':
      // Your code here
    break;
    case 'user_form':
      // Your code here
    break;
  }
}
```

We have set the title displays to `invisible`. We can now modify our `twig` template to apply a visually hidden class (such as Bootstrap's `sr-only`) to labels that are set to `invisible`.

The logic in `form-element-label.html.twig` would look something like this:

~~~
{%-
  set classes = [
    title_display == 'invisible' and not (is_checkbox or is_radio) ? 'sr-only',
  ]
-%}
{% if title is not empty and title_display == 'invisible' and (is_checkbox or is_radio) -%}
  {#
  Clear but preserve label text as attribute (e.g. for screen readers) for
  checkboxes/radio buttons when it actually should be invisible.
  #}
  {%- set attributes = attributes.setAttribute('title', title) -%}
  {%- set title = null -%}
{%- endif -%}
{#
~~~

And finally your a11y class in your css:

```css
// Only display content to screen readers
//
// See: http://a11yproject.com/posts/how-to-hide-content

.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  margin: -1px;
  padding: 0;
  overflow: hidden;
  clip: rect(0,0,0,0);
  border: 0;
}
```

The `user_pass` form is exactly the same as `user_login_form`, however `user_register_form` and `user_form` require a slightly different approach.

Lets hide the labels and add placeholders to `user_register_form`:

```php
/**
 * Implements hook_form_alter().
 */
function mymodule_form_alter(&$form, $form_state, $form_id) {
  switch($form_id) {
    case 'user_login_form':
      $form['name']['#attributes']['placeholder'] = t('Foo');
      $form['pass']['#attributes']['placeholder'] = t('Bar');
      $form['name']['#title_display'] = 'invisible';
      $form['pass']['#title_display'] = 'invisible';
    break;
    case 'user_pass':
      // Your code here
    break;
    case 'user_register_form':
      $form['account']['mail']['#attributes']['placeholder'] = t('E-mail');
      $form['account']['name']['#attributes']['placeholder'] = t('Username');
      $form['account']['mail']['#title_display'] = 'invisible';
      $form['account']['name']['#title_display'] = 'invisible';
    break;
    case 'user_form':
      // Your code here
    break;
  }
}
```

Notice we have to go through `'account'` to get the same result.

`user_form` is similar however requires some extra work to get results.

Let's say we want to change the titles of the `current_pass`, `pass[pass1]` and `pass[pass2]` fields in this form:

```php
/**
 * Implements hook_form_alter().
 */
function mymodule_form_alter(&$form, $form_state, $form_id) {
  switch($form_id) {
    case 'user_login_form':
      $form['name']['#attributes']['placeholder'] = t('Foo');
      $form['pass']['#attributes']['placeholder'] = t('Bar');
      $form['name']['#title_display'] = 'invisible';
      $form['pass']['#title_display'] = 'invisible';
    break;
    case 'user_pass':
      // Your code here
    break;
    case 'user_register_form':
      $form['account']['mail']['#attributes']['placeholder'] = t('E-mail');
      $form['account']['name']['#attributes']['placeholder'] = t('Username');
      $form['account']['mail']['#title_display'] = 'invisible';
      $form['account']['name']['#title_display'] = 'invisible';
    break;
    case 'user_form':
      $form['account']['current_pass']['#title'] = t("Current password");
      $form['#after_build'][] = 'mymodule_after_build';
    break;
  }
}

function mymodule_after_build($form, &$form_state) {
  $form['account']['pass']['pass1']['#title'] = t('New password');
  $form['account']['pass']['pass2']['#title'] = t('Confirm new password');
  return $form;
}
```

Notice the `current_pass` title can be changed with exactly the same approach we took to change the `user_register_form` title display and placeholder. The big difference is where we have had to use `$form['#after_build']` to get access to the `pass[pass1]` and `pass[pass2]` elements after the form elements are built. We can then make whatever changes we want via the `mymodule_after_build` function above.

What if we want to hide the labels and add placeholder text instead of changing the titles? No problem:

```php
/**
 * Implements hook_form_alter().
 */
function mymodule_form_alter(&$form, $form_state, $form_id) {
  switch($form_id) {
    case 'user_login_form':
      $form['name']['#attributes']['placeholder'] = t('Foo');
      $form['pass']['#attributes']['placeholder'] = t('Bar');
      $form['name']['#title_display'] = 'invisible';
      $form['pass']['#title_display'] = 'invisible';
    break;
    case 'user_pass':
      // Your code here
    break;
    case 'user_register_form':
      $form['account']['mail']['#attributes']['placeholder'] = t('E-mail');
      $form['account']['name']['#attributes']['placeholder'] = t('Username');
      $form['account']['mail']['#title_display'] = 'invisible';
      $form['account']['name']['#title_display'] = 'invisible';
    break;
    case 'user_form':
      $form['account']['current_pass']['#attributes']['placeholder'] = t("Current password");
      $form['account']['current_pass']['#title_display'] = 'invisible';
      $form['#after_build'][] = 'memories_after_build';
    break;
  }
}

function mymodule_after_build($form, &$form_state) {
  $form['account']['pass']['pass1']['#attributes']['placeholder'] = t('New password');
  $form['account']['pass']['pass2']['#attributes']['placeholder'] = t('Confirm new password');
  $form['account']['pass']['pass1']['#title_display'] = 'invisible';
  $form['account']['pass']['pass2']['#title_display'] = 'invisible';
  return $form;
}
```

Easy!

So there we have it. We can extend these principles to `/node/edit` and other forms but that's a blog post for another day.

I hope this is of benefit to themers who want to extend their beautiful themes to Drupal 8's authentication and user forms.
