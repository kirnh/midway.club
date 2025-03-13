+++
date = '2025-03-13'
draft = false
title = 'Local Hugo Static Website Setup'

+++

**Step 1:**

**1.1: Install Hugo**

Hugo is a static site generator that is simple to use. Install it based on your OS:

* **macOS (Homebrew)**

  `brew install hugo`

* **Windows (Chocolatey)**

  `choco install hugo -confirm`

* **Ubuntu/Debian**

  `sudo apt install hugo`

**1.2: Create Your Site**

Run:

```
hugo new site dev-docs
cd dev-docs
```

**1.3: Choose a Theme**

Hugo themes define the design of your site. You can pick one from https://themes.gohugo.io/.

For a simple, aesthetic theme, let’s use **Paper**:

```
git init
git submodule add https://github.com/nanxiaobei/hugo-paper themes/paper
```

Enable the theme in config.toml:

`echo 'theme = "paper"' >> config.toml`

Remove hugo.toml if it exists.

**1.4 Add Homepage**

```
mkdir -p content
echo -e "---\ntitle: \"Home\"\ndraft: false\n---\n\nWelcome to my development documentation site!" > content/_index.md
```

**1.5: Add Your First Post**

Hugo uses Markdown for content. Create a page:

`hugo new posts/hugo-static-site-setup-guide.md`

Edit the file content/posts/hugo-static-site-setup-guide.md and add:

```
---
title: "Hugo Static Site Setup Guide"
date: 2025-03-13
---

This is the guide for setting up your site.
```

**1.6: Run the Local Server**

To preview the site:

`hugo server`

Open http://localhost:1313/ in your browser.

---

**Step 2: Host the Site**

To make the site public, we’ll use **GitHub Pages** (free hosting).

**2.1: Create a GitHub Repository**

* Go to [GitHub](https://github.com/) and create a new repository (e.g., dev-docs).

**2.2: Push Your Site to GitHub**

```
git remote add origin https://github.com/yourusername/dev-docs.git
git branch -M main
git add .
git commit -m "Initial commit"
git push -u origin main
```

**2.3: Deploy to GitHub Pages**

1. Install Hugo’s deploy tool:

   `sudo apt install rsync`

2. Run:

   `hugo -d public`

3. Push the public folder to GitHub Pages.

---

**Step 3: Connect Your Domain**

1. Go to your domain registrar (e.g., Namecheap, GoDaddy).
2. Set up a **CNAME record** pointing to GitHub Pages.
3. Wait for DNS propagation.

---

**Next Steps**

* Customize the theme (themes/paper/layouts).
* Add more documents (hugo new posts/your-topic.md).
* Automate deployment with GitHub Actions.
