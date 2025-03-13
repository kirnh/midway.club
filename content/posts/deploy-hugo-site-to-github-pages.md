---
date: 2025-03-13
title: "Deploying Hugo site to GitHub Pages"
---



**🚀 Step 1: Push Your Hugo Site to GitHub**

**1.1: Create a GitHub Repository**

​	1.	Go to [GitHub](https://github.com/) and create a **new repository** named midway-club.

​	2.	**Do not** add a README, .gitignore, or license (we will handle this manually).

​	3.	Copy the repository URL (e.g., https://github.com/yourusername/midway-club.git).

**1.2: Initialize Git in Your Hugo Project**

```
git init
git remote add origin https://github.com/yourusername/midway-club.git
git branch -M main
git add .
git commit -m "Initial commit"
git push -u origin main
```

This pushes your Hugo site to GitHub.

---

**🚀 Step 2: Set Up GitHub Actions for Auto-Deployment**

GitHub Actions will **automatically deploy** your site when you push changes.

**2.1: Create a GitHub Actions Workflow**

Run:

```
mkdir -p .github/workflows
touch .github/workflows/deploy.yml
```

**2.2: Add Deployment Script**

Edit .github/workflows/deploy.yml and add this:

```
name: Deploy Hugo Site

on:
  push:
    branches:
      - main  # Runs on every push to main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4

      - name: Install Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: 'latest'

      - name: Build Site
        run: hugo --minify

      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v4
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
```

**2.3: Commit & Push the Workflow**

```
git add .github/workflows/deploy.yml
git commit -m "Add GitHub Actions for deployment"
git push origin main
```

This will trigger a deployment workflow. You can check the **“Actions”** tab in your GitHub repository to see if it runs successfully.

---

