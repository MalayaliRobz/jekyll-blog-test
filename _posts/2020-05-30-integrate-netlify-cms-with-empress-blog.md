---
layout: post
current: current
class: post-template
subclass: post tag-ember-addon
navigation: true
title: How to integrate Netlify-CMS with Empress-blog
cover: /assets/uploads/ember-netlify1.png
tags: ember netlify-cms empress
author: robin
date: 2020-06-24 01:30:24
---
So you use [Empress-blog](https://github.com/empress/empress-blog) for your blog website. Do you edit and add blogs in local? What if you don't have your  personal laptop? Do you then re-create the whole project just to add a new blog post? What if I told you there was a better way!

> Wouldn't be great if you could have a CMS for your blog and that too free of cost. Add and Edit blog posts from the browser itself. 🤩

Let me introduce you to **[Netlify CMS](https://www.netlifycms.org/)** your open source content management for your Git workflow.

##### OK just show me how!

## 1) Create a new Empress-blog project

Let's first create a new ember project and convert it into a empress-blog project.

Run the following commands in the terminal, in the folder where you'd like to create the blog.

```shell
ember new super-blog
cd super-blog
ember install empress-blog empress-blog-attila-template
```

It will ask you if you want to update the `index.html` file and you should say yes 👍

For this tutorial, we will be using the [empress-attila-template](https://github.com/empress/empress-blog-attila-template).

## Get to know Empress

In your favourite code editor, open up the code generated, and take a look at the `content` directory.

You will see that there are multiple Markdown files that represent blog posts. Open one `.md` file and you will see something like this:

```markdown
---
title: Welcome to empress-blog
image: /images/welcome.jpg
imageMeta:
  attribution:
  attributionLink:
featured: true
authors:
  - ghost
date: Tue Jun 12 2018 18:59:59 GMT+0100 (IST)
tags:
  - getting-started
---

Hey! Welcome to empress-blog, it's great to have you :)
```

We can see above that each blog post has a title, image, author, body and other properties. We will be recreating this using Netlify CMS.

Similarly, have a look at *author* and *tags* folder `.md` files. We will need them for adding author(s) and tag(s) for your blog posts.

**Note:** for other global configuration options, check out [](https://github.com/empress/empress-blog)<https://github.com/empress/empress-blog#configuring>

Try running in local with `npm start`  and your blog should be live in [http://localhost:4200](http://localhost:4200/)

> Great things are working in local. Let's move on to the CMS part.

## 2) Add Netlify CMS to your site

> Update: you don't need to do step 2 anymore. Just `ember install empress-blog-netlify-cms` and you are good to go. For more details check out [https://github.com/MalayaliRobz/empress-blog-netlify-cms](https://github.com/MalayaliRobz/empress-blog-netlify-cms).

 Let's start giving your blog some super powers. In the `public` folder create `admin` folder with `config.yml` and `index.html` files in it.

```
├── public
     ├── admin
         ├ index.html
         └ config.yml
```

In your `config.yml` file paste the following configuration:

```yaml
backend:
  name: git-gateway
  branch: master
media_folder: 'public/uploads'
public_folder: "/uploads"
collections:
  - name: 'blog'
    label: 'Blog'
    folder: 'content/'
    create: true
    slug: "{% raw %}{{year}}-{{month}}-{{day}}-{{slug}{% endraw %}"
    fields:
      - { name: "title", label: "Title", widget: "string" }
      - { name: "image", label: "Image", widget: "image", required: false }
      - name: "imageMeta"
        label: "ImageMeta"
        widget: "object"
        collapsed: true
        fields: 
          - { label: "attribution", name: "attribution", widget: "string", default: '', required: false }
          - { label: "attributionLink", name: "attributionLink", widget: "string", default: '', required: false }
      - { name: "featured", label: "Featured", widget: "boolean", default: true }
      - { name: "authors", label: "Authors", widget: "list", default: ["ghost"] }
      - { name: "tags", label: "Tags", widget: "list", default: ["new"], required: false }
      - { name: "body", label: "Body", widget: "markdown" }
      - { label: "Publish Date", name: "date", widget: "datetime", format: "ddd MMM DD YYYY h:mm:ss a"}

  - name: 'author'
    label: 'Author'
    folder: 'author/'
    create: true
    slug: "{% raw %}{{name}}{% endraw %}"
    fields:
      - { name: "name", label: "Name", widget: "string" }
      - { name: "id", label: "Id", widget: "string" }
      - { name: "image", label: "Image", widget: "image", required: false }
      - { name: "cover", label: "Cover", widget: "string", required: false }
      - { name: "website", label: "Website", widget: "string", required: false }
      - { name: "twitter", label: "Twitter", widget: "string", required: false }
      - { name: "facebook", label: "Facebook", widget: "string", required: false }
      - { name: "location", label: "Location", widget: "string", required: false }
      - { name: "body", label: "Body", widget: "markdown", required: false }

  - name: 'tag'
    label: 'Tag'
    folder: 'tag/'
    create: true
    slug: "{% raw %}{{name}}{% endraw %}"
    fields:
      - { name: "name", label: "Name", widget: "string" }
      - { name: "image", label: "Image", widget: "image", required: false }
      - { name: "body", label: "Body", widget: "markdown", required: false }
```

For this tutorial we will be using git-gateway as our backend for Netlify CMS. More details can be found [here.](https://www.netlifycms.org/docs/backends-overview/)

In your `index.html` file, paste in the following contents.

```html
<!doctype html>
<html>
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Content Manager</title>
  <script src="https://identity.netlify.com/v1/netlify-identity-widget.js"></script>
</head>
<body>
  <!-- Include the script that builds the page and powers Netlify CMS -->
  <script src="https://unpkg.com/netlify-cms@^2.0.0/dist/netlify-cms.js"></script>
</body>
</html>
```

**Some more configuration**

While adding users to your CMS, netlify will send them an email with a link to your blog url containing an access token. We need to setup a watch for this and redirect them to the admin page so that netlify can verify them and add them as users. 

In `index.html` file in the `app` folder. Add `<script src="https://identity.netlify.com/v1/netlify-identity-widget.js"></script>` to the *<head>* tag. Also after the *<body>* tag add this small script.

```html
  <script>
    if (window.netlifyIdentity) {
      window.netlifyIdentity.on("init", user => {
        if (!user) {
          window.netlifyIdentity.on("login", () => {
            document.location.href = "/admin/";
          });
        }
      });
    }
  </script>
```

The whole index.html file should look something like this now.

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    
    <script src="https://identity.netlify.com/v1/netlify-identity-widget.js"></script>

    {% raw %}{{content-for "head"}}{% endraw %}

    <link integrity="" rel="stylesheet" href="{% raw %}{{rootURL}}assets/vendor.css{% endraw %}">
    <link integrity="" rel="stylesheet" href="{% raw %}{{rootURL}}assets/super-blog.css{% endraw %}">

    {% raw %}{{content-for "head-footer"}}{% endraw %}
  </head>
  <body>
    {% raw %}{{content-for "body"}}{% endraw %}

    <script src="{% raw %}{{rootURL}}assets/vendor.js{% endraw %}"></script>
    <script src="{% raw %}{{rootURL}}assets/super-blog.js{% endraw %}"></script>

    {% raw %}"{{content-for "body-footer"}}{% endraw %}
  </body>

  <script>
    if (window.netlifyIdentity) {
      window.netlifyIdentity.on("init", user => {
        if (!user) {
          window.netlifyIdentity.on("login", () => {
            document.location.href = "/admin/";
          });
        }
      });
    }
  </script>
</html>
```

> 🥳 we are almost done. Just a few more steps.

### 3) Push to GitHub

It's now time to commit your changes and push to GitHub. The ember new command does in-fact initialise a Git project automatically for you, so you only need to do:

```shell
git add .
git commit -m "Initial Commit"
git remote add origin https://github.com/YOUR_USERNAME/NEW_REPO_NAME.git
git push -u origin master
```

### 4) Add your repo to Netlify

Go to Netlify and select 'New Site from Git'. Select GitHub and the repository you just pushed to. Click Configure Netlify on GitHub and give access to your repository. Finish the setup by clicking Deploy Site. Netlify will begin reading your repository and starting building your project.

### 5) Enable Identity and Git Gateway

Netlify's Identity and Git Gateway services allow you to manage CMS admin users for your site without requiring them to have an account with your Git host or commit access on your repo. From your site dashboard on Netlify:

1. Go to **Settings > Identity**, and select **Enable Identity service**.
2. Under **Registration preferences**, select **Open** or **Invite only**. In most cases, you want only invited users to access your CMS, but if you're just experimenting, you can leave it open for convenience.
3. If you'd like to allow one-click login with services like Google and GitHub, check the boxes next to the services you'd like to use, under **External providers**.
4. Scroll down to **Services > Git Gateway**, and click **Enable Git Gateway**. This authenticates with your Git host and generates an API access token. In this case, we're leaving the Roles field blank, which means any logged-in user may access the CMS. For information on changing this, check the [Netlify Identity documentation](https://docs.netlify.com/visitor-access/identity/).

## 6) Accessing the CMS

Your site CMS is now fully configured and ready for login!

If you set your registration preference to "Invite only," invite yourself (and anyone else you choose) as a site user. To do this, select the **Identity** tab from your site dashboard, and then select the **Invite users** button. Invited users receive an email invitation with a confirmation link. Clicking the link will take you to your site with a login prompt.

If you left your site registration open, or for return visits after confirming an email invitation, access your site's CMS at `yoursite.com/admin/`.

After logging-in to the admin page, you should see a screen similar to 👇.

![netlify cms dashboard screenshot](/assets/uploads/screenshot-2020-05-30-at-8.35.18-pm.png)

## 7) Start publishing

It's time to create your first blog post. Click on `New Blog` button and start adding the title, image and other properties right from the handy interface. When you click Publish, a new commit will be created in your GitHub repo with this format `Create Blog “year-month-date-title”`.

Then Netlify will detect that there was a commit in your repo and will start rebuilding your project. When your project is deployed you'll be able to see the post you created.

**Note:** to add tags or authors, click on the respective collection name from the side menu and click on the `New` button.

### [](https://www.netlifycms.org/docs/gatsby/#cleanup)Cleanup

It is now safe to remove the default Empress blog posts. You can use the `delete entry` option available in the CMS.

That's it you have successfully connected your Empress-blog with Netlify CMS. 🎉

Github repo: <https://github.com/MalayaliRobz/empress-netlify-cms-starter>

Fore more details on Empress, Netlify CMS check out <https://github.com/empress/empress-blog> and <https://www.netlifycms.org/docs/intro/>