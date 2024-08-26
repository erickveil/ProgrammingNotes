---
title: Excluding an Indexed Directory in Git
layout: note
tags:
  - git
---

# Problem:

I ran `git add -A` and accidentally added a `build` directory. I want to exclude that directory and everything in it (recursively), and I also want to not commit what I just added.

# Solution:

To exclude the `build` directory and everything in it, you should add an entry to your `.gitignore` file and then remove the `build` directory from the staging area. Here's how you can do it:

1. **Add `build` directory to `.gitignore`:**
   Open your `.gitignore` file (or create one if it doesn't exist) and add the following line:
   ```
   build/
   ```

2. **Remove the `build` directory from the staging area:**
   Run the following command to remove the `build` directory from the staging area:
   ```sh
   git rm -r --cached build
   ```

3. **Commit the changes to `.gitignore`:**
   Now commit the changes to the `.gitignore` file and the removal of the `build` directory from the staging area:
   ```sh
   git add .gitignore
   git commit -m "Add build directory to .gitignore and remove it from staging area"
   ```

4. **Reset other changes if needed:**
   If you want to unstage all changes that were added accidentally, you can reset the staging area:
   ```sh
   git reset
   ```

`git reset` is used to unstage changes that were added to the staging area (also known as the index) by the `git add -A` command. Essentially, it resets the staging area to match the most recent commit, undoing any changes that were staged but not yet committed.

Here are the commands all together:

```sh
echo 'build/' >> .gitignore
git rm -r --cached build
git add .gitignore
git commit -m "Add build directory to .gitignore and remove it from staging area"
git reset
```

