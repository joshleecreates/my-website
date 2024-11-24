---
author: Josh Lee
date: "2019-11-19T08:30:53Z"
tags:
- WordPress
title: A Developer's Guide to WordPress
url: /developers-guide-wordpress/
---

Let’s take a look at the most popular CMS on the planet: *WordPress*. If you’re a developer inexperienced with WordPress, some of what you’re about to read will surely make you cringe.

**If you’re a senior engineer who wants to understand the architecture of WordPress, then this guide is for you.**

So let’s pull back the curtain and take a look at some of the architecture in WordPress:

### **Posts — One Table to Rule Them All**

WordPress was initially built for blogging, and so its core model is called the post. In WordPress, almost everything is a post. The built-in post types besides `post` (blog posts) are: `page`, `attachment`, and `link`\*.

Pages are of course exactly what they sound like — static pages displayed on the site without regard for chronological order.

Attachments are used to identify file uploads (which can then be included in other posts).

*(\*) There are also some “hidden” post types used for navigation menus and other content. The `link` post type is deprecated, but was once used for “related blogs.”*

### **Custom Post Types**

Part of what makes WordPress so extensible is this fundamental concept that *everything* is a post. Plugins and themes can define their own custom post types which gain all of the behavior of posts or pages right out of the box. Of course, they can also be further customized from there.

Custom post types were fully introduced in [WordPress 3.0](https://codex.wordpress.org/Version_3.0) released in June 2010. They are one of the features that first attracted me to the WordPress ecosystem.

### **Metadata**

With one table for everything, it would be insanity to attempt to create a column for every possible meta field for every possible post type. Instead, WordPress relies on a `post_meta` table which contains key-value pairs of metadata.

The keys can be anything and the metadata persists even if the plugin that originally created it is uninstalled. User meta is handled similarly.

This provides WordPress a great amount of flexibility for *managing content* and for displaying pages and sections of a website. It also makes WordPress posts much more difficult to index and query against than a normalized SQL database.

If your content management system depends on complex querying based on metadata, you may want to look beyond WordPress… or you may want to integrate a service like Elastic Search or Algolia into your WordPress site.

### **Taxonomies and terms**

In addition to metadata, WordPress posts (and other post types) can belong to any number of terms. Terms belong to a taxonomy. The built in taxonomies are `post_category` and `post_tag`. They are only applied to the post post type, although they can be extended to other post types, including page, through hooks.

Creating custom taxonomies and connecting them to custom or built-in post types is extremely straight forward, requiring [a single function call](https://developer.wordpress.org/reference/functions/register_post_type/).

### **Action Hooks: almost Event Listeners**

Now we get into the meat and potatoes of WordPress customization: action hooks. Action hooks are called at various points in the request lifecycle, as the page is built. They allow callback functions to be injected at essentially any point in the WordPress system.

This is the key to WordPress’ extensibility and also a cause for frequent developer frustration. Action hooks are a bit more finicky than more advanced solutions like pub/sub or event listeners. They have a concept of priority, and they often operate on global variables that are “built up” through a series of callbacks which may lie in multiple plugins.

WordPress includes myriad [built-in action hooks](https://codex.wordpress.org/Plugin_API/Action_Reference). Many well-made plugins will also include their own action hooks so that they can be extended as well as WordPress core. These “extendable extensions” are like a dream come true for developers.

All of this combines to give WordPress unparalleled extensibility amongst CMS platforms.

### **Filters**

In addition to hooks, WordPress provides a number of filters. Filters are essentially action hooks except that the callback function takes an argument (the data being “filtered”) and returns the processed argument.

### **Plugins &amp; Themes**

There are few distinctions in WordPress between plugins and themes. Either one can run any arbitrary PHP at any point after it has been loaded. In the case of themes, this is done through the functions.php file. In plugins it is the main php file for the plugin.

Plugins are loaded just before the theme, however most plugin and theme developers rely on the `init` action hook as the first available action hook to begin customization. This ensures that all plugins and the active theme are fully loaded.

### **The Template Hierarchy**

WordPress is not an MVC framework. It provides the routing layer for you, and then interprets your templates following a [cascading hierarchy](https://developer.wordpress.org/themes/basics/template-hierarchy/) — meaning you can have a custom template for every single category, or even every single post, *or* you can build your entire theme from a single index.php file.

![The Template Hierarchy](https://lh3.googleusercontent.com/EpZcHv9LsSK254glXg5bXMFtWxRxPqhyfYEhk8mvqFvYnGlvYFPk2OxHUCHMeMn9uAePaXO7I5Bv0Bc6uziUcV4Xpz9pfKVSaMAeiSG1bU2udyvBxlZaXndKb_YMOJunjlOn1mW7)

The WordPress template hierarchy, via [Michelle Schulp ☜](https://medium.com/u/5920fb423d45)

### **Blocks &amp; The (Gutenberg) Block Editor**

WordPress 5.0, released in late 2018, introduced the “Block Editor” (also known as the “Gutenberg” editor). Blocks represent a fundamental reimagining of the core WordPress editor, providing a more “WYSIWYG” experience than the classic editor.

Under the hood, the Block Editor uses React.js for the admin editor experience. The content of the blocks is rendered as HTML and saved inside the content of the post or page.

There are several dozen [blocks built into WordPress](https://wordpress.org/support/article/blocks/) and theme and plugin authors can create their own custom blocks. Registering a custom block requires creating a custom javascript file which can be loaded in the admin using the `enqueue_block_editor_assets` hook. This javascript file must call `registerBlockType()` in order to alert WordPress to the blocks’ presence and settings.

For most use cases, you can probably find a plugin that provides the block types you need. You can also use the Advanced Custom Fields plugin to create your own custom block types without writing any javascript or react code.

### **Sidebars &amp; Widgets**

WordPress sidebars are really just “widget buckets” and widgets are simple content blocks that are configured once for all pages on which they appear. I’m not the biggest fan of the way that the sidebars API works, however it is undeniably effective at empowering non-technical users to layout their own websites with pretty much any custom functionality they can imagine.

These can get unwieldy. It is not uncommon to have to go on a plugin and sidebar widget purge after inheriting a WordPress website.

### **Globals everywhere**

WordPress relies on a lot of global variables. In recent years the core team has made a push to move away from the singleton pattern in core APIs. That work is still in progress and the singleton pattern still persists in core as well as in plugins and themes.

In addition, globals are used for most of the content that appears on a page during a typical request, the\_post being the primary of these.

### **Absolute URLs**

WordPress, quite controversially, relies on absolute URLs for all internal linking. On top of this, PHP arrays are serialized in the database, meaning that find-and-replace operations must be performed in PHP on the deserialized data, rather than directly in MySQL with queries. These two factors make migrating WordPress websites cumbersome relative to simpler PHP platforms.

### **Why is WordPress so successful?**

Clearly, despite some of the potential anti-patterns employed, WordPress is quite successful, powering [~20% of the websites on the internet](https://managewp.com/14-surprising-statistics-about-wordpress-usage). As with anything WordPress’s success is multi-factorial; I would argue that it boils down to three things, some of which are actually served by what could be considered anti-patterns:

- **Accessibility.**
    One of the core principles of WordPress is to always cater to the least-technically savvy as much as possible. While many of us have heard horror stories, really it is quite remarkable that WordPress makes it possible for regular users to install any of the plugins available and build their own websites.  
    *This is because even when implementing developer-oriented features, the WordPress core developers carefully consider the impact on their millions of end-users.  
    On top of this, WordPress provides a gentle curve from non-technical writer to semi-competent developer; I’ve seen myriad users progress from editing a few lines of CSS to building complete custom plugins.

- **Extensibility.** 
    As I noted while discussing hooks, WordPress is immensely extensible, to the point that even it’s plugins can be extended with other plugins. Many of the anti-patterns in the core code persist because of the WordPress community’s commitment to backwards compatibility with the approximately 46 *thousand* plugins available.

- **The Right Opinion at the Right Time** WordPress was born with a core thesis about website structure: a website is fundamentally a collection of posts and pages, and everything can be boiled down to one of those two types. This strong opinion provided a shared foundation on top of which the many plugins could be developed with impressive interoperability.

### **When should you use it?**

Well, that depends.

If you’re just trying to put a few static pages online, you may want to consider a static website builder, but WordPress could also work well.

If you need a simple eCommerce website, you may want to compare Woocommerce (*the* WordPress plugin for eCommerce) and Shopify.

Clearly, it is purpose-built for [blogging](https://joshuamlee.com) and a great solution for that.

Marketers are often already familiar with WordPress, and it provides integrations with most marketing automation and analytics services.

If you need a content-oriented website with multiple authors and editors, WordPress is also a clear choice: Content marketing and WordPress go together hand-in-hand.

- - - - - -

While WordPress may not be for everybody or every situation, I hope I have shown that it has merit when applied correctly. Fear not, all hope is not lost.

#### **Further Reading:**

- [The Codex](https://codex.wordpress.org/) (Older documentation)
- [Code Reference](https://developer.wordpress.org/reference/) (Newer documentation, less complete)
- [WordPress Slack](https://make.wordpress.org/chat/)
- [Make WordPress](https://make.wordpress.org/) – handbook and resources for core contributors
- [Coding Standards](https://make.wordpress.org/core/handbook/best-practices/coding-standards/php/) – style guides for PHP and CSS
