---
title: "Modern Blogging: Hugo, GitHub Pages, and a Custom Domain"
date: 2026-03-23
description: "A guide for IT enthusiasts: From local installation on OpenBSD to automated deployment via GitHub Actions."
tags: ["Hugo", "GitHub Pages", "OpenBSD", "DevOps"]
categories: ["Tutorials"]
featuredImage: "/images/hugo-github-setup.png"
---

Do you have a project idea and are looking for a simple way to present it on your own website? With **Hugo** and **GitHub Pages**, you can get it done in no time. Using this site as an example, I’ll show you how straightforward it is to build your own presence.

## Requirements

The barriers to your “own website” project are minimal. All you need is:

* **GitHub account**: This is where your code will be managed and published.
* **Custom domain (optional)**: If you want your site accessible via a personal address.
* **Operating system with a terminal**: A solid command line is essential (this guide uses **OpenBSD** as an example).

## Why Hugo?

For someone who works a lot in the terminal (and appreciates minimalism as an OpenBSD maintainer), Hugo is a blessing. It generates static HTML, which means:

* **Security:** No SQL injections or PHP vulnerabilities.
* **Speed:** The site loads almost instantly.
* **Portability:** All content is stored in Markdown files.

## Workflow on OpenBSD

Installing Hugo, Git, and some additional useful tools is very simple thanks to available packages:

```shell
doas pkg_add hugo git nvi ImageMagick
```

## Choosing a Theme

Before getting started, you have plenty of options: Hugo offers a huge selection of themes. In the [official overview](https://themes.gohugo.io/) you can explore designs and test them in live demos.

In this guide, I use the [LoveIt theme](https://themes.gohugo.io/themes/loveit/) as an example.

{{< admonition type="info" open=true >}}
**Good to know**: Thanks to Hugo's strict separation of content and design, you can generally switch themes at any time. However, please note that each theme uses its own variables in the configuration file. Switching later may therefore require some manual adjustments to adapt the settings to the new theme.
{{< /admonition >}}

## Preparations

### DNS Configuration (optional)

Detailed information can be found in the [GitHub documentation](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site).

For a personal portfolio page, creating a user page is the easiest way:

1. **Create repository**: Name it `[your-username].github.io`
2. **DNS config**: Add a CNAME record pointing to your GitHub page

{{< admonition type="info" open=true >}}
If you don't have your own domain (yet), your site will be accessible at [Your-GitHub-Username].github.io.
{{< /admonition >}}

### Git(-Hub) Settings

Configure the following settings in GitHub to publish your site:

* **Create Repository**: If you haven't already, create a new repository using the naming scheme `[Your-GitHub-Username].github.io`.

* **Configure GitHub Pages**: Navigate to the Pages menu item in your repository settings and enter the following details:

  * **Build and deployment**: Under *Source*, select the **GitHub Actions** option.

  * **Custom Domain**: If you want to use your own domain, enter your hostname (e.g., `buzzdeee.reitenba.ch`) and confirm with **Save**. The DNS check should be successfully completed after a few moments.

  * **Enforce HTTPS**: Enable this checkbox to ensure that your site is encrypted and accessible exclusively via HTTPS.

{{< admonition type="info" open=true >}}
**Note**: The checkbox *Enforce HTTPS* is deactivated in the beginning, it will become active **after** first run of the workflow.
{{< /admonition >}}


Your settings should then look like the following example:

{{< figure
    src="/images/GitHub_Pages_Settings.png"
    alt="GitHub Pages configuration settings"
    caption="Example configuration of GitHub Pages settings"
    class="ma0 w-75"
>}}

##Minimal Setup and Initialization

Once you have decided on a theme, we will lay the foundation for your project.

First, create a local directory, switch into it, initialize a new Git repository, and add the **LoveIt theme** as a submodule. This has the advantage that you can easily keep the theme up to date later without mixing it with your own code.

{{< admonition type="info" open=false >}}
**Create and enter directory**
```
mkdir buzzdeee.github.io
cd buzzdeee.github.io
```
**Initialize Git repository (with 'main' as the default branch)**
```
git init -b main
```

**Add and initialize theme as a submodule**
```
git submodule add https://github.com/dillonzq/LoveIt.git themes/LoveIt
git submodule update --init --recursive
```
{{< /admonition >}}

## Adjusting the Configuration

To avoid starting from scratch, we copy the theme's example configuration into our root directory and edit it using an editor (e.g., `nvi` or `vi`):

{{< highlight shell "linenos=table,open=true" >}}
cp themes/LoveIt/exampleSite/hugo.toml .
nvi hugo.toml
{{< /highlight >}}

In the `hugo.toml` file, you should primarily check and individualize the following points:

* **BaseURL**: The final address where your site will be accessible (e.g., https://buzzdeee.reitenba.ch/).
* **Author Information**: Name, social links, and a short bio.
* **Language Settings**: Default language and menu entries for navigation.
* **Comment Function**: For the start, it is recommended to deactivate this (a detailed setup would be beyond the scope of this guide).

Below you will find the most important configuration examples to serve as a guide for your own `hugo.toml`:

{{< highlight toml "linenos=table,open=false" >}}
baseURL = "https://buzzdeee.reitenba.ch"

# theme
theme = "LoveIt"
# themes directory
themesDir = "themes"

defaultContentLanguage = "de"

[languages]
  [languages.de]
    weight = 1
    languageCode = "de"
    languageName = "Deutsch"
    title = "Mein Blog"
    [languages.de.menu]
      [[languages.de.menu.main]]
        name = "Posts"
        url = "posts"
        weight = 1
      [[languages.de.menu.main]]
        weight = 2
        identifier = "tags"
        pre = ""
        post = ""
        name = "Tags"
        url = "/tags/"
        title = ""
      [[languages.de.menu.main]]
        weight = 3
        identifier = "categories"
        pre = ""
        post = ""
        name = "Kategorien"
        url = "/categories/"
        title = ""
      [[languages.de.menu.main]]
        weight = 4
        identifier = "documentation"
        pre = ""
        post = ""
        name = "Docs"
        url = "/categories/documentation/"
        title = ""
      [[languages.de.menu.main]]
        weight = 5
        identifier = "about"
        pre = ""
        post = ""
        name = "About"
        url = "/about/"
        title = ""
      [[languages.de.menu.main]]
        weight = 6
        identifier = "github"
        pre = "<i class='fab fa-github' aria-hidden='true'></i>"
        post = ""
        name = ""
        url = "https://github.com/myaccount"
        title = "GitHub"
  [languages.en]
    weight = 2
    languageCode = "en"
    languageName = "English"
    title = "My Blog"
    [languages.en.menu]
      [[languages.en.menu.main]]
        name = "Posts"
        url = "posts"
        weight = 1
      [[languages.en.menu.main]]
        weight = 2
        identifier = "tags"
        pre = ""
        post = ""
        name = "Tags"
        url = "/tags/"
        title = ""
      [[languages.en.menu.main]]
        weight = 3
        identifier = "categories"
        pre = ""
        post = ""
        name = "Categories"
        url = "/categories/"
        title = ""
      [[languages.en.menu.main]]
        weight = 4
        identifier = "documentation"
        pre = ""
        post = ""
        name = "Docs"
        url = "/categories/documentation/"
        title = ""
      [[languages.en.menu.main]]
        weight = 5
        identifier = "about"
        pre = ""
        post = ""
        name = "About"
        url = "/about/"
        title = ""
      [[languages.en.menu.main]]
        weight = 6
        identifier = "github"
        pre = "<i class='fab fa-github' aria-hidden='true'></i>"
        post = ""
        name = ""
        url = "https://github.com/buzzdeee"
        title = "GitHub"
...
  # Author config
  [params.author]
    name = "buzzdeee"
    email = ""
    link = ""
...
[params.page.comment]
    enable = false
...
{{< /highlight >}}

{{< admonition type="tip" title="Multilingual Support" open=true >}}
Store your articles as Markdown files in the `./content/posts/` directory. For Hugo to recognize that two files represent the same content in different languages, they must have the identical filename and only differ by the language code.

Example:

* `my-article.de.md` (German version)
* `my-article.en.md` (English version)

Only then can visitors switch directly between the languages of the respective article.
{{< /admonition >}}

## The First Local Test Run

Once the configuration is set, it's time for the first trial run. Start the Hugo server with the command `hugo server -D` (the `-D` flag ensures that drafts are also displayed):

{{< admonition type="error" title="Startup Error" open=false >}}
```
ERROR error building site: failed to acquire a build lock:
open /home/user/GIT/buzzdeee.github.io/.hugo_build.lock: too many open files
```
{{< /admonition >}}

**Ouch!** On OpenBSD, you often run into a classic limit problem (`too many open files`). This happens because Hugo monitors many files simultaneously by default.

There are two ways to solve this:

1. **The Clean Solution (Increase system limits)**: Increase the values for `openfiles-max` and `openfiles-cur` in the `/etc/login.conf` file to `4096`. After logging in again, `hugo server -D` should start without issues, including automatic file monitoring (watch mode).
2. **The "Quick Fix" (Without file monitoring)**: If you don't want to change the system configuration, you can start Hugo with reduced features. This disables automatic reloading upon changes:

{{< highlight shell "linenos=table" >}}
hugo server --disableFastRender --ignoreCache --noHTTPCache --watch=false
{{< /highlight >}}

Once the server is running successfully, you can view and test your intermediate result in your browser at `http://localhost:1313/` in real-time.

## The Avatar Image

There are two ways to provide the profile picture for your home page:

* **Gravatar**: A global service that provides your image linked to your email address.
* **Local Image**: An image stored directly in your repository.

### Pros and Cons of Gravatar

* **Pro**: You only need to update your image in one central place ([gravatar.com](https://gravatar.com)), and it will automatically be updated across all linked platforms (GitHub, StackOverflow, your blog, etc.).

* **Con (Privacy)**: To load the image, the visitor's browser sends a request to Gravatar's servers. This transmits a hash of your email address and the visitor's IP address, which can be sensitive regarding privacy (GDPR).

### Avatar Configuration in hugo.toml

Depending on which path you choose, you either enter your email address at `gravatarEmail` or adjust the path at `avatarURL`:

{{< admonition type="info" title="Configuration Example" open=true >}}
```
    [params.home.profile]
      enable = true
```

**Gravatar email for the profile picture (optional)**
```
      gravatarEmail = ""
```
Local path to the profile picture
```
      avatarURL = "/images/avatar.png"
```
{{< /admonition >}}

{{< admonition type="tip" title="Storage Location for Images" open=true >}}
Static content, such as images, belongs in the `./static/` directory. The avatar image should therefore be stored at `./static/images/avatar.png.` Hugo then makes it available at `/images/avatar.png` during the build process.
{{< /admonition >}}

## Favicon

To make your site look good in the browser tab, you need a favicon. The `favicon.ico` file belongs directly in the `./static/` directory.

{{< admonition type="tip" title="Convert Avatar to Favicon" open=true >}}
You can use the convert tool from the ImageMagick package to easily transform your avatar image into a suitable favicon:
```
convert static/images/avatar.png -define icon:auto-resize=64,48,32,16 static/favicon.ico
```
{{< /admonition >}}

## Creating the "About" Page

To ensure the "About" menu item doesn't lead to a 404 error, we need to create the corresponding content file. Since Hugo distinguishes between regular blog posts and static pages, we place this file directly in the `content` folder.

### Creating the Files

Create a separate Markdown file for each language under `./content/about/`:

* **German**: `./content/about/index.de.md`

* **English**: `./content/about/index.en.md`

Content Example (`about/index.en.md`)

{{< highlight markdown "linenos=table,open=false" >}}
title: "About me"
layout: "single"
nodateline: true
noauthor: true

Here you can briefly introduce yourself, describe your projects, or explain what this blog is about. Since this file is named index.en.md, it will automatically be displayed under the URL /about/.
{{< /highlight >}}

{{< admonition type="tip" title="Multilingual Linking" open=true >}}
Just like with blog posts, Hugo recognizes that these pages belong together based on the identical path/filename. If a visitor is on the "About me" page and switches the language, they will land directly on the German counterpart `about/index.de.md`.
{{< /admonition >}}

{{< highlight markdown "linenos=table,open=false" >}}
title: "About me"
layout: "single"
nodateline: true
noauthor: true

Here you can briefly introduce yourself, describe your projects, or explain what this blog is about. Since this file is named index.en.md, it wi
ll automatically be displayed under the URL /about/.
{{< /highlight >}}

{{< admonition type="tip" title="Multilingual Linking" open=true >}}
Just like with blog posts, Hugo recognizes that these pages belong together based on the identical path/filename. If a visitor is on the "About
me" page and switches the language, they will land directly on the German counterpart `about/index.de.md`.
{{< /admonition >}}

## Creating Your First Post

Now that the framework is ready, it's time for some real content. To create a new blog post, it’s best to use the `hugo new` command. This creates the file with the appropriate basic structure (the so-called Frontmatter).

### Step 1: Create Files

In your terminal, create the German and English versions of your post one after the other:

{{< highlight shell "linenos=table" >}}
hugo new posts/my-first-post.de.md
hugo new posts/my-first-post.en.md
{{< /highlight >}}

### Step 2: Set Status to "Published"

By default, Hugo marks new posts as drafts. To make them appear on your live site, you must either delete the line `draft = true` in the file header (Frontmatter) or set it to `false`:

{{< highlight markdown "linenos=table" >}}
title: "My First Post"
date: 2026-03-25T10:00:00+01:00
draft: false
{{< /highlight >}}

{{< admonition type="tip" title="Draft Mode during Testing" open=true >}}
As long as `draft = true` is set, you will only see the post locally if you start the server with the `-D` flag (`hugo server -D`). On the final website on GitHub, drafts are ignored.
{{< /admonition >}}

## Automation with GitHub Actions

To ensure your site is automatically built and published with every `git push`, we use *GitHub Actions*. To do this, create the file `.github/workflows/hugo.yaml` in your repository and insert the following content:

{{< highlight yaml "linenos=table,open=false" >}}
name: Deploy Hugo site to Pages

on:
  push:
    branches: ["main"]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

defaults:
  run:
    shell: bash

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: recursive # to include the theme
          fetch-depth: 0

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v3
        with:
          hugo-version: 'latest'
          extended: true # LoveIt theme requires extended version (SCSS)

      - name: Build with Hugo
        run: hugo --minify

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
{{< /highlight >}}

### Activation in the Repository Settings

To ensure that GitHub actually uses this workflow (if you haven't done so already), you need to make a small change in the settings:
Navigate to **Settings** > **Pages**. Under *Build and deployment*, select the **GitHub Actions** option from the *Source* dropdown menu.


## Keeping It Clean: The .gitignore File

When testing and building your site locally, temporary files and directories (such as the `public/` folder) are created that do not belong in your Git repository. To exclude these, create a file named `.gitignore` in your root directory:

{{< admonition type="info" title=".gitignore" open=true >}}
**Local build directory (will be regenerated on GitHub)**
```
public/
```
**Resource cache and temporary files**
```
resources/
.hugo_build.lock
```
{{< /admonition >}}

## The Final Git Push

Everything is now prepared to take your site live. Use the following commands to transfer your local configuration to your GitHub repository:

{{< highlight shell "linenos=table" >}}
git add .
git commit -m "Initial commit of the Hugo site"
git remote add origin git@github.com:buzzdeee/buzzdeee.github.io.git
git push -u origin main
{{< /highlight >}}

### Potential Hurdle: GitHub Push Protection

It may happen that GitHub rejects the push with an error message. This is usually because the LoveIt theme's example configuration contains a default key for **Mapbox**, which GitHub blocks for security reasons:

{{< admonition type="error" title="Error: git push rejected" open=false >}}
```
remote: error: GH013: Repository rule violations found for refs/heads/main.
remote: - GITHUB PUSH PROTECTION
remote: Push cannot contain secrets
remote: Mapbox Secret Access Token found in hugo.toml:472
```
{{< /admonition >}}

**The Solution**: In this specific case, it is a known demo key from the theme. You have two options:

1. Delete the key in `hugo.toml` (around line 472) if you are not using Mapbox.

2. Follow the link in the error message (`https://github.com/.../unblock-secret/...`) to inform GitHub that this specific key should be ignored.

Once the key has been cleared or removed, the `git push` will complete successfully, and the build process will start automatically.

{{< admonition type="tip" title="Pro Tip: Safety First" open=true >}}
If you use your own real API keys later, **never** write them directly into the `hugo.toml`. Instead, use environment variables or **GitHub Actions Secrets**. This keeps your credentials private and prevents them from ending up in your repository's public code.
{{< /admonition >}}

## Conclusion & Checklist

Congratulations! If everything went well, your personal website is now accessible worldwide under your own domain. Thanks to Hugo and GitHub Actions, you now have an extremely fast, secure, and low-maintenance system.

Before you start writing your next article, here is a quick **Success Checklist**:

* [ ] **Domain reachable?** Check if your URL (e.g., `https://buzzdeee.reitenba.ch`) resolves correctly.
* [ ] **HTTPS active?** Does the padlock icon appear in your browser's address bar?
* [ ] **Language switch okay?** Does switching between German and English work on the "About" page and in your posts?
* [ ] **Favicon visible?** Is your custom icon displayed in the browser tab?
* [ ] **GitHub Action green?** Check the *Actions* tab in your GitHub repo to see if the build completed without errors.

## What's Next?

You can now create new content at any time by simply creating a new Markdown file and uploading it via `git push`. Automation will handle the rest for you.

Please note that this guide only covers the basic framework. The LoveIt theme offers many more powerful features that I haven't covered in detail here to keep things concise. These include:

* **Mapbox Integration**: For interactive maps.

* **Comment Systems**: Integration of Disqus, Utterances, or Giscus.

* **Social Media**: Linking your profiles in the navigation or footer.

* **Shortcodes**: Special Hugo commands for galleries, music players, or YouTube videos.

The further journey of discovery through the [LoveIt Documentation](https://hugoloveit.com/) and trying out new features is now entirely up to you.

**Good luck and have fun with your new blog!**
