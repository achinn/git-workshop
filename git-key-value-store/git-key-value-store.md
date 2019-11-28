# Git is a key-value store

- Keys are SHA-1 hashes
- Values are referred to as "objects"
- Backing store is the file system
- **DO:** Create a new repo
  `git init <name>`
- **DO:** Check out the file system
  `ls .git/objects/`

---

# Writing objects

- **DO:** Create an object
  `echo -n '<file contents>' | git hash-object -w --stdin`
- `hash-object` generates a key and writes to the object store if given `-w`
- **DO:** What does the file system look like now?
  `ls .git/objects/`
- **DO:** Create two objects with the same contents. What are the keys?

---

# Reading objects

- **DO:** Read from the object store
  `git cat-file -p <hash>``
- **DO:** Examine type of object
  `git cat-file -t <hash>`

---

# How objects are stored

```
content = "<file contents>"
header = "blob <content length>\0" (null terminated)
key = SHA-1(header + content)
file_contents = ZlibDeflate(header + content)
file_path = ".git/objects/" + key[0,2] + "/" + key[2,38]
WriteFile(file_path, file_contents)
```

---

# How objects are stored

- **DO:** Implement this in `irb`
  `require 'digest/sha1'; require 'zlib'; require 'fileutils`
  See `Digest::SHA1.hexdigest`
  See `Zlib::Deflate.deflate`
  See `File.dirname`, `FileUtils.mkdir_p`, `File.write`
- **DO:** Verify it worked
  `git cat-file`

---

Sketch out your current mental model of the object store

![fit](box.png)

---

# Hierarchy among objects

- Problem: Now I have to keep track of hashes for my objects?
- Another type of object: tree
- Trees point to both blobs and trees but have human friendly names for what they point to
- Blob objects are leaf nodes

---

# Writing trees

- **DO:** Create a tree object
  `echo -en "100644 blob <hash>\t<name>" | git mktree`
- Input is of the format
  `"<file mode> <object type> <hash>\t<name>"`
- `mktree` generates a key as a hash on the object list and writes a tree to the object store

---

# Writing trees

- **DO:** Figure out how to make a tree pointing to multiple blobs
  Hint: the input kind of looks like a file listing doesn't it?
- **DO:** Figure out how to make a tree pointing to sub-trees
  File mode for trees is `040000`
- **DO:** Verify the types and inspect the trees with `cat-file`

---

# How trees are stored

- Same as for blobs except
  `content = "<file mode> <name>\0<bytes of object hash>"`
- **BONUS DO:** Implement this in `irb`
  See `Array#pack` with a argument of `'H*'`

---

Sketch out your current mental model of the object store

![fit](box.png)

---

# Enter a third object type

- Continued problem: Now I have to keep track of hashes for my trees?
- Yet another type of object: commits
- A commit points to a tree and an optional parent commit
- The tree pointed to represents a snapshot
- Commits have metadata: author, committer, timestamp, description

---

# Writing commits

- **DO:** Create a commit object
  `echo "my description" | git commit-tree <hash>`
- `commit-tree` generates a key as a hash on all the inputs and metadata
- Pass in `-p <hash>` option to specify a parent commit

---

# Writing commits

- **DO:** Create a commit pointing to another tree with a commit as a parent
- **DO:** Verify the types and inspect the commits with `cat-file`

---

# How commits are stored

- Same as for blobs except
```
content = "tree <hash>\n
author <author> <timestamp>\n
committer <committer> <timestamp\n\n
<commit description>"
```

---

# What just happened?

- **DO:** See what Git sees
`git show <last commit hash>`
- **DO:** See what the file system sees
`find .git/objects -type f`

---

Sketch out your current mental model of the object store

![fit](box.png)

---

# Recap so far
- Git is a key-value store
- Key: SHA-1 hash of object content and a header
- Value: objects of different types - blob, tree, commit
- Continued problem: I still have to keep track of my last commit hash?

---

# Commit tracking

- We've been writing out objects as files with deterministic, yet non-human-friendly file names
- Let's do the inverse to help out our feeble minds

---

# Creating references

- **DO:** Create a reference
  `git update-ref refs/heads/<name> <hash>`
- **DO:** See what this created
  `cat .git/refs/heads/<name>`
- Human friendly names, stored as files in a well defined place

---

# Listing References

- **DO:** List all references and what commits they point to
  `git show-ref`
- **DO:** We know these as branches
  `git branch`
- Problem: If there are multiple references, which one is the current one?

---

# Symbolic reference

- Reference: a file containing a commit hash
- Symbolic reference: a file containing a reference name
- **DO:** Examine the symbolic reference
  `cat .git/HEAD`
- **DO:** Update the symbolic reference
  `git symbolic-ref HEAD refs/heads/<ref>``

---

Sketch out your current mental model of the object store

![fit](box.png)

---

# ✋ Stop and synchronize with the rest of the group 🛑

---

# Set up (and review)

- **DO:** `git init test && cd test && echo "test" > test.txt`
- **DO:** Create a blob object using `test.txt`
- **DO:** Create a tree containing the blob using "`test.txt`" as the name
- **DO:** Create a commit pointing to the tree
- **DO:** Create a reference pointing to the commit
- **DO:** Update the symbolic reference to point to the reference

---

# The man in the middle

- **DO:** What does `git status` show?
- Conclusion: `HEAD <--> ? <--> Working directory`
- **DO:** Try this out
  `git update-index --add --cacheinfo 100644,<blob hash>,test.txt`
- **DO:** Now what does `git status` show?

---

# The index

```
HEAD <--> Index <--> Working directory
```

- The index is a binary file stored at `.git/index`
- Our `update-index` call made the index look like the last commit of the current branch (`HEAD`)

---

# Viewing the index

```
git ls-files -s
```

- The index keeps track of files related to the current commit
  - `HEAD` -> reference -> commit -> tree

---

# Object store <- Index <- Working directory

- The index also keeps track of changes to files you want to eventually commit
- So far we've been going around the index and straight to the object store
- Normally you'd update the object store _through_ the index

```
echo "test2" > test2.txt && git add test2.txt
```

- Q: What does the index look like after?
- Q: What does the object store look like after? (Hint: use `find`)

---

# Object store <- Index <- Working directory (cont'd)

```
git commit -m "Added test2.txt"
```

- Q: What does the object store look like after?

---

# Object store -> Index -> Working directory (checkout)

```
git checkout HEAD~
```

- Q: What does `.git/HEAD` look like?
- `checkout` updates `HEAD`
- Q: What do the index and working directory look like?
- `checkout` also updates the index and working directory

---

# Object store -> Index -> Working directory (reset)

- `checkout` back to your reference (use ref name) and see that the world is good again

```
git log
```

- We'll need these commit hashes in a bit

```
git reset --soft HEAD~
```

---

# Object store -> Index -> Working directory (reset)

- Q: What does `.git/HEAD` look like?
- Q: What does your reference look like? See `show-ref`.
- `reset` dereferences `HEAD` and updates what that reference points to
- Q: What do the index and working directory look like?
- `--soft` leaves the index and working directory intact

---

# Object store -> Index -> Working directory (reset)

- `reset --soft` back to the commit that your ref used to point to (hash from `git log`) and see that the world is good again

```
git reset --mixed HEAD~
```

----

# Object store -> Index -> Working directory (reset)

- Q: What does `.git/HEAD` look like? What does your reference look like?
- Q: What do the index and working directory look like?
- `--mixed` only leaves the working directory intact
- `--mixed` is the default option for `reset`

---

# Object store -> Index -> Working directory (reset)

- `reset --mixed` back to the commit that your ref used to point to and see that the world is good again

```
git reset --hard HEAD~
```

----

# Object store -> Index -> Working directory (reset)

- Q: What does `.git/HEAD` look like? What does your reference look like? What does your index look like?
- Q: What does the working directory look like?
- `--hard` doesn't leave anything intact

---

# Recap: Levels of change

```
HEAD  <->  reference  <->  index  <->  working directory
          |-------------- add; commit -----------------|
|--------------------- checkout -----------------------|
          | (s)reset |
          |---- (m)reset ------|
          |---------------- (h)reset ------------------|
```

- Bonus: `checkout` and `reset` (mixed) can be used at the file level. Try to figure out how they differ.
