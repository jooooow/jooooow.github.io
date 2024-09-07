---
title: "Git Cheatsheet"
layout: post
---

# Brief
This blog aims to memo some useful but not daily used commands.

# Reference
[progit](https://www.progit.cn)

# Commands

## 1. Untract staged files
```bash
git rm -r --cached .
git add .
...
```

## 2. Rename a file

```bash
git mv file_from file_to
```

this command is basically equivalent to : 

```bash
mv file_from file_to
git mv file_from
git add file_to
```

## 3. Add tag
+ add tag to current commit

```bash
git tag -a tag_name -m tag_message
```

+ add tag to history commit

```bash
git tag -a tag_name -m tag_message commit_hash
```

+ push tag to remote

```bash
git push remote_name tag_name
```





<script src="https://utteranc.es/client.js"
        repo="jooooow/jooooow.github.io"
        issue-term="pathname"
        theme="github-light"
        crossorigin="anonymous"
        async>
</script>