---
layout: post
title: "Useful Shell Commands"
category: shell-command-tricks
---
## Table of Contents
{% toc %}

---

## Print your folder structure in readable format using the `tree` command
Here’s how to do it on Windows:

### **Using Command Prompt (CMD)**
1. Open **Command Prompt**.
2. Navigate to your `mysite` folder:
   ```bash
   cd path\to\mysite
   ```
3. Run the `tree` command to print the folder structure:
   ```bash
   tree /F
   ```
   The `/F` option lists the files in each directory.

   This will output the directory structure to the command line. You can copy the output from there.


### **Example Output**
```plaintext
mysite/
├── _config.yml
├── _layouts/
│   ├── default.html
├── german-language/
│   ├── index.md
│   ├── 2025-01-18-german-grammar-tips.md
├── trips/
│   ├── index.md
│   ├── 2025-01-01-trip-to-rome.md
├── personal-growth/
│   ├── index.md
│   ├── 2025-01-15-building-better-habits.md
```

After running the command, you can simply copy and paste the output.
