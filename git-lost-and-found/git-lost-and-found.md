# git always be writing

1. `git init <name> && cd <name>`
2. Add and commit a large file: `cp /usr/share/dict/words .`
3. `chmod +w` and append a new line of text to the file
4. Add and commit the change.

---

# git always be writing (cont'd)

- What was written to the object store?
  - `git ls-tree` to find blob objects
  - `du -ah` to see file sizes in the object store

---

# Packing it up

```
git gc
```

---
