# Agenda

1. What is the Git object store?
2. How do do we track things in the object store?
3. What is the Git index and how does it interact with the object store?

---

# Logistics

- **DO:** describes a task
- `<...>` is a placeholder for your own value
- **Q:** asks a question for you to answer. Write down your answer
- **REC:** indicates to copy some value to a text editor
- `echo`, `cat`, `find` are vanilla Unix commands

---

# Git is a key-value store

- Keys are SHA-1 hashes
- Values are referred to as "objects"
- **DO:** Create a new repo
  `git init <name> && cd <name>`

---

# Writing objects

- **DO:** Create an object
  `echo -n "<file contents>" | git hash-object -w --stdin`
- `git hash-object` generates a key and writes to the object store if given `-w`
- **REC:** The hash that was returned
- **DO:** The backing store is the file system. See what it looks like
  `find .git/objects -type f`

---

# Reading objects

- **DO:** Read from the object store
  `git cat-file -p <hash>`
- **DO:** Examine the type of the object
  `git cat-file -t <hash>`
- **Q:** Create two objects with the same contents. What are the keys?

---

# How objects are stored

- Pseudocode:
```
1. object_content = "<file contents>"
2. header = "<object type> <object_content length>\0"
3. key = SHA-1(header + object_content)
4. file_contents = ZlibDeflate(header + object_content)
5. file_path = ".git/objects/" + key[1,2] + "/" + key[3,38]
6. WriteFile(file_path, file_contents)
```
Note: `"\0"` is a null terminator

---

Sketch out your current mental model of the object store

---

# Hierarchy among objects

- Another type of object: tree
- Trees point to blobs and other trees
- Tress have human friendly names for what they point to

---

# Writing trees

- **DO:** Create a tree object pointing to your blob object. Pick any name for the blob
  `echo -en "100644 blob <blob hash>\t<blob name>" | git mktree`
- **REC:** The tree hash that was returned
- Input is of the format
  `"<file mode> <object type> <object hash>\t<object name>"`
  - File mode is very much like Unix file modes (user permissions)

---

# Writing trees

- **DO:** Make another blob. Then make a tree pointing to both blobs
  `echo -en "100644 blob <hash1>\t<name1>\n100644 blob <hash2>\t<name2>" | git mktree`
- **DO:** Make a tree pointing to the previous tree
  `echo -en "040000 tree <tree hash>\t<tree name>" | git mktree`
- **REC:** The two tree hashes that were returned
- **DO:** Inspect the object types and values
 `git cat-file -t <tree hash>; git cat-file -p <tree hash>`

---

# How trees are stored

- **Q:** What does the object store look like?
  `find .git/objects -type f`
- See *How objects are stored* (slide 6). Trees are created using the same algorithm except:

```
1. object_content = "<file mode> <name>\0<blob or tree hash>"
2. header = "tree <object_content length>\0"
...
```
- Note: as you saw there can be multiple objects in `object_content`

---

Sketch out your current mental model of the object store

---

# Enter a third object type

- Yet another type of object: commits
- A commit points to a tree and an optional parent commit
- Commits have metadata: author, committer, timestamp, description

---

# Writing commits

- **DO:** Create a commit object using one of your tree objects
  `echo "<desc>" | git commit-tree <tree hash>`
- **REC:** The commit hash that was returned

---

# Writing commits

- **DO:** Create a commit using another tree and the previous commit as a parent
  `echo "<desc>" | git commit-tree -p <commit hash> <tree hash>`
- **REC:** The commit hash that was returned
- **DO:** Inspect the object type and value
 `git cat-file -t <commit hash>`
 `git cat-file -p <commit hash>`

---

# How commits are stored

- **Q:** What does the object store look like?
  `find .git/objects -type f`
- See *How objects are stored* (slide 6). Commits are created using the same algorithm except:

```
1. object_content = "tree <tree hash>
      parent <commit hash>
      author <author> <timestamp>
      committer <committer> <timestamp>

      <commit description>"
2. header = "commit <object_content length>\0"
...
```

---

# What just happened?

- **DO:** See what Git thinks about your commits
  `git log <last commit hash>`
- **Q:** What does Git think happened between each commit?
  For each commit hash from the `log` run
  `git show <commit hash>`

---

Sketch out your current mental model of the object store

---

# Recap so far
- Git is a key-value store
- Value: objects of different types - blob, tree, commit
- Key: SHA-1 hash of object content and a header
  - Object content format varies with the object type

^ Reiterate this verbally

---

# ✋ Stop and synchronize with the rest of the group 🛑

---

# Commit tracking

- Keeping track of commit hashes is hard
- We've been writing out objects as files with deterministic, yet non-human-friendly file names
- Let's do the inverse to help out our feeble minds
- Introducing *references* - human friendly file name pointing to commit hash

---

# Creating references

- **DO:** Create a reference to your last commit
  `git update-ref refs/heads/<name> <commit hash>`
- **DO:** See what this created
  `cat .git/refs/heads/<name>`
- Human friendly names, stored as files in a well defined place

---

# Listing References

- **DO:** List all references and what commits they point to
  `git show-ref`
- **DO:** Surprise! We know these as branches
  `git branch`

---

# Using references

- A reference can be used in the place of a commit hash
- **DO:** Create a new commit using your reference as a parent
  `echo "<desc>" | git commit-tree -p <ref> <tree hash>`
- **DO:** What is the reference pointing to?
  `git log <ref>`
- **DO:** Update your reference to point to the new commit
  `git update-ref refs/heads/<ref> <newest commit hash>`
- **DO:** Now what is the reference pointing to?
  `git log <ref>`

---

# Reference tracking

- You may have many references
- Let's add another level of indirection to keep track of a "current" reference
- Introducing the *symbolic reference* - a file containing a reference name (typically)
  - As opposed to a regular reference which is a file containing a commit hash
  - There is only one of these

---

# Symbolic reference

- **DO:** Examine the symbolic reference
  `cat .git/HEAD`
- **DO:** Update the symbolic reference
  `git symbolic-ref HEAD refs/heads/<ref>`
- **DO:** Examine the symbolic reference
  `cat .git/HEAD`

---

# Using the symbolic reference

- Just like references, the symbolic reference can be used in a place of a commit hash
- **DO:** What is the current commit?
  `git log HEAD`
- **DO:** We can also use a relation to the current commit
  `git log HEAD~`
  `git log HEAD~2`

---

Sketch out your current mental model of the object store

---

# ✋ Stop and synchronize with the rest of the group 🛑

---

# Set up

- **DO:** Create a new repo
  `git init test && cd test`
- **DO:** Create a new file in the working directory
  `echo "test" > test.txt`
- **DO:** Create a blob object using `test.txt` (no `--stdin`)
  `git hash-object -w test.txt`
- **DO:** Create a tree with the blob using "`test.txt`" as the name
  `echo -en "100644 blob <blob hash>\ttest.txt" | git mktree`

---

# Set up

- **DO:** Create a commit pointing to the tree
  `echo "First commit" | git commit-tree <tree hash>`
- **DO:** Create a reference pointing to the commit
  `git update-ref refs/heads/my-branch <commit hash>`
- **DO:** Update the symbolic reference to point to the reference
  `git symbolic-ref HEAD refs/heads/my-branch`

---

# The man in the middle

- **Q:** What does `git status` show?
- It seems something is out of sync between the object store and the working directory
- **DO:** Try this out
  `git update-index --add --cacheinfo 100644,<blob hash>,test.txt`
  Note: there are no spaces between the commas
- **Q:** Now what does `git status` show?

---

# The index

- The index keeps track of files related to the current commit
- The index also keeps track of changes to files you want to eventually commit
- We've been going _around_ the index, straight to the object store
- Normally you'd update the object store _through_ the index
- `git update-index` updated the index to match what `HEAD` sees
- **DO:** It's a binary file at `.git/index`. Inspect with
  `git ls-files -s`

---

Sketch out your current mental model of how the object store, index and working directory relate to each other

---

# Using the index

- **DO:** Add another file
  `echo "test2" > test2.txt && git add test2.txt`
- **Q:** What does the index look like after?
  `git ls-files -s`
- **Q:** What does the object store look like w.r.t. the new file?
  `find .git/objects -type f`
- **DO:** Create a commit for the current changes
  `git commit -m "Added test2.txt"`
- **Q:** What does the object store look like now?


---

# HEAD -> Index -> Working directory (checkout)

- **DO:** Perform a checkout of the previous commit
  `git checkout HEAD~`
- **Q:** What does the message say?
- **Q:** What does the symbolic ref look like?
  `cat .git/HEAD`
- **Q:** What does your reference look like?
  `git show-ref`
- **Q:** What do the index and working directory look like?
  `git ls-files -s` and `ls`

---

# Remember those hashes

- **DO:** Set the symbolic ref back
  `git checkout <your reference name>`
- **REC:** The commit hashes from `git log`. We'll need these for later

---

# HEAD -> Index -> Working directory (soft reset)

- **DO:** Perform a soft reset to the previous commit
  `git reset --soft HEAD~`
- **Q:** What do the symbolic ref and your reference look like?
  `git log`
- **Q:** What do the index and working directory look like?
  `git ls-files -s` and `ls`

^
- `reset` dereferences `HEAD` and updates what that reference points to
- `--soft` leaves the index and working directory intact

---

# Did we just lose our commit?

- **DO:** Is your commit still in the object store?
  `find .git/objects -type f`
- **DO:** Go back to the commit that your ref used to point to
  `git reset --soft <commit hash>`


---

# HEAD -> Index -> Working directory (mixed reset)

- **DO:** Perform a mixed reset to the previous commit
  `git reset --mixed HEAD~`
- **Q:** What do the symbolic ref and your reference look like?
  `git log`
- **Q:** What do the index and working directory look like?
  `git ls-files -s` and `ls`
- `--mixed` is the default option for `reset`

^
- `--mixed` only leaves the working directory intact

---

# Did we just lose our commit?

- **DO:** Is your commit still in the object store?
  `find .git/objects -type f`
- **DO:** Go back to the commit that your ref used to point to
  `git reset --mixed <commit hash>`

----

# HEAD -> Index -> Working directory (hard reset)

- **DO:** Perform a hard reset to the previous commit
  `git reset --hard HEAD~`
- **Q:** What do the symbolic ref and your reference look like?
  `git log`
- **Q:** What do the index and working directory look like?
  `git ls-files -s` and `ls`

^
- `--hard` doesn't leave anything intact

---

# Did we just lose our commit?

- **DO:** Is your commit still in the object store?
  `find .git/objects -type f`
- **DO:** Go back to the commit that your ref used to point to
  `git reset --hard <commit hash>`

---

# Recap

- **DO:** Indicate the levels of change for the various commands
  - `add` combined with `commit` has been done for you

|   | Object store | HEAD | Reference | Index | Working dir |
|---|:---:|:---:|:---:|:---:|:---:|
| `add; commit` | ☑️ | | ☑️ | ☑️ | |
| `checkout` | | | | | |
| `reset --soft` | | | | | |
| `reset --mixed` | | | | | |
| `reset --hard` | | | | | |


^
|   | Object store | HEAD | Reference | Index | Working dir |
|---|---|:---:|:---:|:---:|:---:|
| `checkout` | | ☑️ | | ☑️ | ☑️ |
| `reset --soft` | | | ☑️ | | |
| `reset --mixed` | | | ☑️ | ☑️ | |
| `reset --hard` | | | ☑️ | ☑️ | ☑️ |

---

# Realistic commit recovery

- Fortunately we ran `git log` and recorded commit hashes before messing with our reference. It allowed us to get our work back, especially in the hard reset case.
- But what if we didn't?
- **Q:** What do you see by running `git reflog`?

---

# Realistic commit recovery

- What if the hash you want isn't in the `reflog` for some reason?
- **DO:** Hard reset to the previous commit and delete the `reflog`
  ```
  git reset --hard HEAD~
  rm -rf .git/logs/
  git reflog
  ```
- **DO:** We can still locate orphaned objects in the object store
  `git fsck --full`
