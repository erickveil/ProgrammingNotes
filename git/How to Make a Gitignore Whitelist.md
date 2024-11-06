---
title: How to Make a Gitignore Whitelist
layout: note
tags:
  - git
---

It is possible to track only whitelisted files in a Git repository by using the `.gitignore` file, along with some Git commands. 

1. **Initialize your repository** (if you haven't already):

    ```bash
    git init
    ```

2. **Create a `.gitignore` file** to ignore all files:

    ```bash
    echo "*" > .gitignore
    ```

3. **Create a whitelist of files** that you want to track. For example, if you want to track `file1.txt` and `dir/file2.txt`, you would add these exceptions to your `.gitignore` file:

    ```bash
    echo "!file1.txt" >> .gitignore
    echo "!dir/" >> .gitignore
    echo "!dir/file2.txt" >> .gitignore
    ```

4. **Make sure to add any parent directories** of whitelisted files to the `.gitignore` file, prefixed with `!`, as shown in the previous step for `dir/`.

5. **If you already have files in your repository** that are being tracked, you need to remove them from the index before applying the new `.gitignore` rules. To do this, run:

    ```bash
    git rm -r --cached .
    ```

    This will untrack all files in your repository.

6. **Add the whitelisted files back to the index**:

    ```bash
    git add .
    ```

    This will only add the files that match your whitelist patterns.

7. **Commit the changes** to finalize the setup:

    ```bash
    git commit -m "Setup whitelisted files tracking"
    ```

Here is an example of what your `.gitignore` file might look like:

```
# Ignore all files
*

# Whitelist specific files and directories
!file1.txt
!dir/
!dir/file2.txt
```

