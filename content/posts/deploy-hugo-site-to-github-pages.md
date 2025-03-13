----

date = '2025-03-13'
draft = false
title = 'Deploying Hugo site to GitHub Pages'

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