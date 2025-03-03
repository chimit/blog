---
title: Don't use @lang directive in Blade templates
author: Chimit
pubDatetime: 2025-03-03T12:00:00Z
featured: false
draft: false
tags:
  - Laravel
  - Blade
  - XSS
  - vulnerability
  - security
description: "Why you should avoid using the @lang directive in Laravel Blade templates."
---

Laravel has a very powerful template engine called Blade. One of the features of Blade is the `@lang` directive, which is used to translate text strings. I've been using it a lot in my projects and always thought it's the recommended way of using localization in Blade templates. But recently I discovered that it's not mentioned anywhere in the Laravel documentation anymore. In this article, I'll explain why you should be careful with `@lang` and how to use it properly.

## Table of contents

## Why @lang is not safe

The `@lang` directive is a convenient way to translate text strings in Blade templates. It looks like this:

```blade
@lang('messages.welcome')
```

This will output the translated string for the given key. The problem with this approach is that it doesn't escape the translated string. Sometimes it may be a desired behavior, e.g. if your translated string contains HTML tags:

```php
// resources/lang/en/messages.php
return [
    'welcome' => 'Welcome to <strong>My Website</strong>!',
];
```

But if the translated string contains user input, it can lead to XSS vulnerabilities. For example, if the `$name` includes some HTML/JavaScript code like `<script>alert('XSS')</script>`, it will be executed:

```php
// resources/lang/en/messages.php
return [
    'greeting' => 'Hello, :name!',
];
```

```blade
<!-- $name is not escaped ❌ -->
@lang('messages.greeting', ['name' => $name])
```

## How to use @lang safely

To avoid XSS vulnerabilities, you should always escape user input when using the `@lang` directive. You can do this by using the `e` helper function:

```blade
<!-- $name is escaped ✅ -->
@lang('messages.greeting', ['name' => e($name)])
```

Another way is to use the `__` helper function together with Blade's double curly braces that always escape the output automatically:

```blade
<!-- $name is escaped ✅ -->
{{ __('messages.greeting', ['name' => $name]) }}
```

This way is not that expressive as the `@lang` directive, but it's what Laravel documentation recommends since Laravel 8 so I personally prefer to use it.

## Escaping user input without escaping HTML tags in translation strings

In the situations when your translation string contains legit HTML tags, you should escape only the user input:

```php
// resources/lang/en/messages.php
return [
    'greeting' => 'Hello, <strong>:name</strong>!',
];
```

```blade
<!-- $name is escaped ✅ -->
@lang('messages.greeting', ['name' => e($name)])

<!-- $name is escaped ✅ -->
{!! __('messages.greeting', ['name' => e($name)]) !!}
```

I hope this article helps you to write more secure Laravel applications. If you have any questions or feedback, feel free to leave a comment below.
