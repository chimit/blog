---
title: "How to manage AI prompts in Markdown in Laravel"
author: Chimit
pubDatetime: 2025-09-28T10:00:00Z
featured: false
draft: false
tags:
  - Laravel
  - Blade
  - AI
  - Prompts
  - Markdown
description: "Effectively manage AI prompts in Markdown using the power of Blade for better organization and maintainability."
---

In my work on [Cointry](https://cointry.io), I frequently interact with AI and prompts. As the application grows, managing these prompts becomes increasingly challenging. Embedding prompts directly as strings within a controller or service can quickly lead to messy, hard-to-maintain code riddled with concatenated strings, embedded variables, and conditional logic, all within a single PHP method.

```php
$productName = 'Wireless Headphones';
$productDescription = 'Amazing sound quality...';

$prompt = "You are an SEO expert. Generate a meta description for this product: {$productName}. Description: {$productDescription}. Make it compelling and under 160 characters.";
```

This works for a simple case. But when the prompt gets more complex, several issues arise:

- **It gets messy:** You start adding more variables, conditional logic (`if ($product->on_sale)`), and loops. PHP code quickly becomes cluttered with text generation logic.
- **It's hard to maintain:** It's not easy to review or edit prompts without touching the code.
- **No version history:** The prompt is just a piece of a larger code file. You can't easily track how the prompt has evolved over time.
- **No Markdown highlighting:** The prompt is just a string in PHP. You miss out on the readability and formatting benefits of Markdown.

## Prompts as Views

The first thing we need to do to improve our code maintainability is to separate the prompt from the code. And here two concepts come to mind: language files and views. Both of them support passing variables and have some level of organization. But language files are not really designed for large blocks of text with formatting, and they don't support complex logic well. Views, on the other hand, are perfect for this use case.

Laravel has a powerful templating engine called Blade. It supports variables, conditionals, loops, and even raw PHP code if needed. This makes it an excellent fit for managing AI prompts. However, Blade works with `.blade.php` files but I wanted to use Markdown for my prompts as it's standard for prompts. So I made a tiny package called [`chimit/prompt`](https://github.com/chimit/prompt) that leverages Blade for this purpose.

### Why is it better?

#### 1. Dynamic and Flexible Prompts

Here is an example of a Markdown prompt file to generate a product meta description:

```markdown
You are an SEO expert specializing in e-commerce. Generate a compelling meta description for this product.

**Product:** {{ $product->name }}
**Price:** ${{ number_format($product->price, 2) }}

## **Product Description:**

## {!! $product->description !!}

@if($product->discount_percentage > 0)
**Special Offer:** {{ $product->discount_percentage }}% OFF - Limited Time!
@endif

Requirements:

- Maximum 160 characters
- Include the product name and key benefits
- Create urgency if there's a discount
- Target keywords: {{ implode(', ', $keywords) }}
```

This is clean, readable, and incredibly dynamic. The logic for the prompt is _in the prompt_, not cluttering PHP code.

Usage is simple and familiar to every Laravel developer:

```php
use Chimit\Prompt;

$prompt = Prompt::get('seo.product-meta', [
    'product' => $product,
    'keywords' => ['wireless headphones', 'bluetooth', 'noise cancelling']
]);
```

#### 2. Clean, Organized, and Maintainable

Prompts get their own dedicated home in `resources/prompts`. You can organize them into subdirectories, just like views.

```
resources/
└── prompts/
    ├── seo/
    │   └── product-meta.md
    └── welcome.md
```

This separation of concerns makes the codebase significantly cleaner. PHP code is responsible for the _logic_ (fetching data, calling the AI), and prompt files are responsible for the _text_.

#### 3. Version Control for Prompts

Since each prompt is a separate file, you can track its history with Git and see who changed a prompt, when, and why. You can create branches to experiment with new prompt ideas without affecting the main application.

#### 4. Markdown highlighting

Markdown is de-facto standard for writing prompts. By using Markdown files, we get syntax highlighting and formatting benefits in the editor, making prompts easier to read and write.

## Conclusion

The [chimit/prompt](https://github.com/chimit/prompt) package provides a much-needed organizational layer for managing AI prompts in a Laravel application. By leveraging the power and familiarity of Blade, it helps to create prompts that are more dynamic, maintainable, and collaborative.
