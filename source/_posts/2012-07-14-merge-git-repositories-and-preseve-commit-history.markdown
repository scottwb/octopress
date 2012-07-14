---
layout: post
title: "Merge Git Repositories and Preseve Commit History"
date: 2012-07-14 11:42
comments: true
categories:
- git
---
Every time I want to combine two git repositories into a new high-level repository, I have to re-figure it out. I decided this time to write it down. Maybe someone else will find this useful too.

Here is my scenario: I have two repositories. I want to make a new empty repository and move the other two into it as subdirectories. I also want to preserve all the commit history of the original repositories.

Here are the steps involved for the first repository:

1. Clone the first source repository
2. Remove its origin ref so you dont' accidentally push any screwups
3. Make a subdir in that source repo named the same as the repo
4. Move all the top-level files into that directory
5. Commit that move.
6. Create (or clone) your desination repository.
7. Add a remote ref to it that points to the file location of your source repo clone.
8. Pull from that ref
9. Remove that remote ref.

You can now delete the clone of the source repository you, so you don't really keep those file moves around in the original source if you don't want to.

Here's what those steps might look like:

``` bash
# Step 1
$ git clone SOURCE_REPO_URL
$ cd source_repo_name

# Step 2
$ git remote rm origin

# Step 3
$ mkdir source_repo_name

# Step 4
#
# NOTE: You can't actually do this command verbatim. You need to
#       git mv each top-level file/dir individually and don't forget
#       .* files like .gitignore, but DO NOT copy the .git directory.
$ git mv * source_repo_name/.

# Step 5
$ git commit -m "Prepare source_repo_name to merge into dest_repo_name"
$ cd ..

# Step 6
$ git clone DST_REPO_URL
$ cd dest_repo_name

# Step 7
$ git remote add source_repo_name SRC_REPO_FILEPATH

# Step 8
$ git pull source_repo_name master

# Step 9
$ git remote rm source_repo_name
```

Now you're done and can delete the source repository clone, and push the destination repository clone upstream. Check the `git log` to be sure.

### A Concrete Example

Say I have two repositories on github named `homer` and `bart` and I want to combine them into a new repository called `simpsons`. Here is how that looks:

``` bash
# ...First create a new empty repository on github named 'simpsons'...

# Clone all the repos to work with
$ cd ~/src
$ git clone git@github.com:scottwb/homer.git
$ git clone git@github.com:scottwb/bart.git
$ git clone git@github.com:scottwb/simpsons.git

# Copy over homer to simpsons
$ cd homer
$ git remote rm origin
$ mkdir homer
$ git mv *.* homer/.
$ git commit -m "Prepare homer to merge into simpsons"
$ cd ../simpsons
$ git remote add homer ~/src/homer
$ git pull homer master
$ git remote rm homer
$ cd ..
$ rm -rf homer

# Copy over bart to simpsons
$ cd bart
$ git remote rm origin
$ mkdir bart
$ git mv *.* bart/.
$ git commit -m "Prepare bart to merge into simpsons"
$ cd ../simpsons
$ git remote add bart ~/src/bart
$ git pull bart master
$ git remote rm bart
$ cd ..
$ rm -rf bart

# Push simpsons back upstream to github
$ cd homer
$ git push
```

### Bonus Points: Only Moving a Sub-Directory

What if you only want to move a subdirectory of the original source repository into the destination repository? All you need to do is filter out what you copy into that new sub-directory you create in step 4, and make sure everything else gets removed from that source repository. (Don't worry - remember, you're just working with a local working copy of that source repo, that you're going top discard after this operation. You won't harm anything irreversibly here.)

One way to peform that filtering is by using the `git filter-branch` command. For example, to only copy the `pranks` subdir from the `bart` repo, just before step 4, you would do something like this:

``` bash
$ git filter-branch --subdirectory-filter pranks -- --all
```

That dumps all the contents of the `pranks` dir into top-level dir where you can proceed to move them into your new subdir that you created in step 3.
