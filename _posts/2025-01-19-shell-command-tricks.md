---
layout: post
title: "Script Notes"
category: shell-command-tricks
---

There are so many shell commands I keep forgetting or just can’t seem to remember. This is my go-to spot to store all of them, so I can easily come back when I need them — and you can too, if you ever need one!

## Table of Contents
1. [Print your folder structure in readable format](#print-your-folder-structure)
   - [Using Command Prompt (CMD)](#using-command-prompt-cmd)
      - [Example Output](#example-output)
2. [Redirect standard error to standard output](#redirect-standard-error-to-standard-output)
      - [Example](#example)
   - [Alternate Syntax](#alternate-syntax-for-compatibility)
      - [Example](#example-1)

---

## **Print your folder structure**
Print your folder structure in readable format using the `tree` command. Here’s how to do it on Windows:

### Using Command Prompt (**CMD**)
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


#### Example Output
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

---

## **Redirect standard error to standard output**
To redirect the standard error (`stderr`) to the same place as the standard output (`stdout`), you use the `&>` operator. Here's how you can do it:

```bash
command &> output.txt
```
- `command`: The command you want to execute.
- `&>`: Redirects both `stdout` (file descriptor 1) and `stderr` (file descriptor 2) to the same location.
- `output.txt`: The file where both `stdout` and `stderr` will be written.

#### Example:
```bash
ls non_existing_file &> output.txt
```

- If `non_existing_file` does not exist, the error message will be written to `output.txt`.

### **Alternate Syntax (for Compatibility)**
If you're using older shells that do not support `&>`, you can use this:

```bash
command > output.txt 2>&1
```

- `> output.txt`: Redirects `stdout` to `output.txt`.
- `2>&1`: Redirects `stderr` (file descriptor 2) to wherever `stdout` (file descriptor 1) is going.

#### Example:
```bash
ls non_existing_file > output.txt 2>&1
```

This will have the same effect as `&>` but is more widely supported.

---
