# Git Internals

## [Plumbling and Porcelain](https://git-scm.com/book/en/v2/Git-Internals-Plumbing-and-Porcelain)

* Git is a content-addressable filesystem with a VCS user-interface on top of
  it.
* In older versions, git did not have a user-friendly VCS user-interface and
  only had low-level commands that has to be chained together using a UNIX
  script.
* The low-level command are called "plumbing" commands.
* The user-friendly commands are called "porcelain" commands.

* `git init` &rarr; creates `.git` directory.
```
.git/
  config -> project specific config options
  description -> only used by GitWeb program.
  HEAD -> points to the branch you currently have checked out.
  hooks/ -> Git Hooks.
  info/ -> keeps a global exclude file for ignored patterns that you don’t want
    to track in a .gitignore file.
  objects/ ->  stores all the content for your database.
  refs/ -> stores pointers into commit objects in that data (branches, tags,
    remotes and more).
```

## [Git Objects](https://git-scm.com/book/en/v2/Git-Internals-Git-Objects)

* At the core, Git is a simple key-value data store.
* Insert any kind of content into a Git repository, for which Git will hand you
  back an unique key you can use later to retrieve that content.
* `git hash-object` -> plumbing command.
  * Takes some data, stores it in your `.git/objects` directory and gives you
    back the unique key that now refers to that data object.
  *
    ```sh
    $ git init test
    $ cd test
    $ echo "Hello World" | git hash-object -w --stdin
    557db03de997c86a4a028e1ebd3a1ceb225be238 -> SHA-1 hash
    $ find .git/objects -type f
    .git/objects/55/7db03de997c86a4a028e1ebd3a1ceb225be238
      -> subdirectory is the first two char of the SHA-1 hash and the file name
         is the remaining.
    ```
* `git cat-file`
  * Examine the content of an object.
  * Swiss army knife for inspecting git objects.
  * 
    ```sh
    $ git cat-file -p 557db03de997c86a4a028e1ebd3a1ceb225be238
    Hello World
    ```
*
  ```sh
  $ echo "foobar" > test.txt
  $ git hash-object -w test.txt
  323fae03f4606ea9991df8befbb2fca795e648fa
  $ echo "barfoo" > test.txt
  $ git hash-object -w test.txt
  34a3ec23bc3494205555f0e49d8d1a12ba2fdae8
  $ find .git/objects -type f
  .git/objects/34/a3ec23bc3494205555f0e49d8d1a12ba2fdae8
  .git/objects/32/3fae03f4606ea9991df8befbb2fca795e648fa
  .git/objects/55/7db03de997c86a4a028e1ebd3a1ceb225be238
  $ git cat-file -p 323fae03f4606ea9991df8befbb2fca795e648fa
  foobar
  $ git cat-file -p 34a3ec23bc3494205555f0e49d8d1a12ba2fdae8
  barfoo
  ```
  * These objects are blob type.

### Tree Objects
* Solves the problem of sotring the filename and also allows you to store
  group of files together.
* All content are stored as tree and blob objects.
* Tree corresponds to UNIX directory entries.
* blobs correspond more or less to inodes or file contents.
*
  ```sh
  $ git add test.txt
  $ git commit -m "First Commit"
  $ git cat-file -p main^{tree} # main^{tree} specifies tree object pointed by
    the last commit on your main branch.
  100644 blob 34a3ec23bc3494205555f0e49d8d1a12ba2fdae8	test.txt
  $ git cat-file -p da6b95fdcd5eece1a935496f4b22cae1e1522738
  100644 blob 34a3ec23bc3494205555f0e49d8d1a12ba2fdae8	test.txt
  ```
* Git creates a tree by taking the state of your staging area or index and
  writing a series of tree objects from it.
* `git update-index` & `git write-tree`
  * To create a tree object, you first have to set up an index by staging some
    files.
  *
    ```sh
    $ cat test.txt
    barfoo
    $ git cat-file -p 323fae03f4606ea9991df8befbb2fca795e648fa # old version
    foobar 
    $ git update-index --add --cacheinfo 100644 \
      323fae03f4606ea9991df8befbb2fca795e648fa test.txt

    100644 -> Normal file.
    323fae03f4606ea9991df8befbb2fca795e648fa -> foobar blob.
    test.txt -> file path.

    $ git write-tree
    92bc19e8253db7d1067e7d53bbbe31a3c7ef9897
    $ git cat-file -p 92bc19e8253db7d1067e7d53bbbe31a3c7ef9897
    100644 blob 323fae03f4606ea9991df8befbb2fca795e648fa	test.txt
    ```
  *
    ```sh
    # Creating a new tree with the latest test.txt object & another file.
    $ echo 'new file' > new.txt
    $ git update-index --cacheinfo 100644 \
      34a3ec23bc3494205555f0e49d8d1a12ba2fdae8 test.txt
    $ git update-index --add new.txt
    $ git write-tree
    1444de37cc21d4d492b1e1a84022bea16443a4ed
    $ git cat-file -p 1444de37cc21d4d492b1e1a84022bea16443a4ed
    100644 blob fa49b077972391ad58037050f2a75f74e3671e92	new.txt
    100644 blob 34a3ec23bc3494205555f0e49d8d1a12ba2fdae8	test.txt
    ```
  *
    ```sh
    # We can technically link the first tree under the second tree.
    $ git read-tree --prefix=bak 92bc19e8253db7d1067e7d53bbbe31a3c7ef9897
    $ git write-tree
    c66d62dafe67ff5799ded7d8ca6ddac892bb584c
    $ git cat-file -p c66d62dafe67ff5799ded7d8ca6ddac892bb584c
    040000 tree 92bc19e8253db7d1067e7d53bbbe31a3c7ef9897	bak
    100644 blob fa49b077972391ad58037050f2a75f74e3671e92	new.txt
    100644 blob 34a3ec23bc3494205555f0e49d8d1a12ba2fdae8	test.txt
    $ git cat-file -p 92bc19e8253db7d1067e7d53bbbe31a3c7ef9897
    100644 blob 323fae03f4606ea9991df8befbb2fca795e648fa	test.txt
    ```

### Commit Objects
* With the tree objects, you don't have information on who saved the snapshot,
  when they were saved, or why they were saved. The commit object stores these
  information.
* `git commit-tree`
  * ```
    $ echo "First Commit" | git commit-tree c66d62dafe67ff5799ded7d8ca6ddac892bb584c
    ec06774a53bf300ced8c3d20d351f1feef5f18de
    $ git cat-file -p ec0677
    tree c66d62dafe67ff5799ded7d8ca6ddac892bb584c
    author Aravind Vasudevan <xxx@yyy.com> 1668643005 +0000
    committer Aravind Vasudevan <xxx@yyy.com> 1668643005 +0000

    First Commit

    # Create following commits linking with previous commit
    $ echo "Second Commit" | git commit-tree 92bc19 -p ec0677
    c35ab058fb5893ceca2e2e38e6379185abe6ba7a
    $ echo "Third Commit" | git commit-tree da6b95 -p c35ab0
    5f5ee9da2465c80c4149c189f2db7fbd16882da3

    # Check git history for this commit
    $ git log --stat 5f5ee9
    commit 5f5ee9da2465c80c4149c189f2db7fbd16882da3
    Author: Aravind Vasudevan <aravindvasudev@google.com>
    Date:   Thu Nov 17 00:04:14 2022 +0000

        Third Commit

    test.txt | 2 +-
    1 file changed, 1 insertion(+), 1 deletion(-)

    commit c35ab058fb5893ceca2e2e38e6379185abe6ba7a
    Author: Aravind Vasudevan <aravindvasudev@google.com>
    Date:   Thu Nov 17 00:03:27 2022 +0000

        Second Commit

    bak/test.txt | 1 -
    new.txt      | 1 -
    test.txt     | 2 +-
    3 files changed, 1 insertion(+), 3 deletions(-)

    commit ec06774a53bf300ced8c3d20d351f1feef5f18de
    Author: Aravind Vasudevan <aravindvasudev@google.com>
    Date:   Wed Nov 16 23:56:45 2022 +0000

        First Commit

    bak/test.txt | 1 +
    new.txt      | 1 +
    test.txt     | 1 +
    3 files changed, 3 insertions(+)
    ```
* Essentially this is what git does underneath with `git add` & `git commit`.
* It stores blobs for the files that have changed, updates the index, writes out
  trees, and writes commit objects that reference the top-level trees and the
  commits that came immediately before them.
* Three main Git objects: blob, tree, commit.
