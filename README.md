# ltatakis.com

![Jekyll-Mono Home Page](/images/ss.png)


### Customise Jekyll-Mono

So now your blog is live with its default settings. Let's customise it now.

Edit the _config.yml and enter your site name and description. You can easily turn on Google Analytics tracking, Disqus commenting and cool loking social icons here too.

Jekyll-Mono also comes with the option of setting up the color scheme of your blog. You can do this by editing _variables.scss that lies inside the _sass folder. You can change the main theme color by simply replacing the current hex color value of `$mono` with the one of your choice. A few main theme sample colors are included in there as comments. Apart from the main theme color you can also change the header link color, navbar hover color, hyperlink color as well as the color of the various headings.

A look at the customisations you can do with _variable.scss

```
// Main theme colors 
// Some cool main theme colors(violet:#8476ad;blue:#5cacee;red:#ff7373,#ff6f69;green:#6acf64,#2ddbb3;orange:#ffa268)

$mono-color:#8476ad;                // main theme color(header, links, footer icons, buttons, post-title)
$hl-color: $darkGray;              // header link color (author name and posted on date) for blog post meta 
$navbar-hover-color:$gray;        // navbar hover color (site name and navbar links hover color)
$link-color: $darkerGray;        // normal hyperlink color other than the ones above.


// Heading colors
// You can play around with these too!
$h1-color: $mono-color;
$h2-color: $mono-color; 
$h3-color: $darkerGray; 
$h4-color: $gray;
``` 
Have a look at Jekyll-Mono with four different main theme colors.
 
![Jekyll Mono in 4 different colors](/images/ss-color.png)

Finally you need to set up your avatar. Pick up your avatar, resize it to 220x220px simply using paint or any editor of your choice and upload it to the images folder. Now open _variables.scss, you'll see something like this `$avatar: "/images/avatar.jpg";`. Here change the avatar.jpg to what you have uploaded just now.


### Start Blogging

Publish your first blog post by editing /_posts/2016-03-06-Eternal-Lorem-Ipsum.md. [Markdown Cheatsheet](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet) might come in handy while writing your blog posts in Markdown. If you are not comfortable with writing in Markdown you can use [Prose](http://prose.io/) for writing your blog posts. [This](https://developmentseed.org/blog/2012/june/25/prose-a-content-editor-for-github/s) will help you in setting up prose.

![Post](/images/post.PNG)

To create a post just click on create new file button in /_posts/. Create a new file with the following naming convention: **year-month-day-title.md**. Also make sure to include the front-matter at the top of each new blog post.

You can see the front matter at the top of /_posts/2016-03-06-Eternal-Lorem-Ipsum.md. It looks something like this. You have to change the title and author according to your post. The layout will remain the same.

```
--- 
layout: post
title: The Eternal Lorem Ipsum Placeholder Text Here
author: Author Name
---

```

### About and Projects Page

In the parent folder you'll find about.md and projects.md. These are templates for helping you to set up your biodata/CV and projects that will be available at yourusername.github.io/about and yourusername.github.io/projects respectively. Again you can edit it in Markdown on GitHub or using Prose as per your choice.

