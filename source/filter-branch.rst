..  _filter_branch:

.. index::
    pair: filter; branch
    single: git; filter-branch

Filter branch
=============


References
----------

-   `GitHub Help - Removing sensitive data from a repository
    <https://help.github.com/articles/removing-sensitive-data-from-a-repository/>`_
    gives a consise summary of the task undergone in the following chapter,
    previously it was using ``filter-branch``, but by now it is using
-   The main reference is :gitdoc:`git documentation: filter-branch
    <git-filter-branch.html>`
-   It is introduced Pro-Git `Git Tools - Rewriting History
    <https://git-scm.com/book/en/v2/Git-Tools-Rewriting-History>`_

Keep your repository safe
-------------------------

.. warning::

   This page was written, at a time where filter branch was the only way to
   remove or modify an object all over the history of a repository. But as
   outlined below it is a very dangerous operation, and you should by now, in
   most use cases you should prefer `git-filter-repo`_ or `BFG repo cleaner`_.

This chapter describe some uses of ``git filter-branch`` before using it you
should be aware that this command is destructive, and even the untouched
commits end up with different object names so your new branch is separate from
the original one.

Before trying to use it you should carefully read the
:gitdoc:`Man page warning <filter-branch.html#_warning>` and it's
:gitdoc:`Security section <filter-branch.html#SECURITY>`.

``filter-branch`` and it's alternatives destroy the repository history, if ever
you repository was shared, anyone downstream is forced to manually fix their
history, by rebasing all their topic branches over the new HEAD.  More details
in :gitdoc:`git-rebase(1) - recovering from upstream rebase
<git-rebase.html#_recovering_from_upstream_rebase>`

When filtering branches the original refs, are stored in the namespace
``refs/original/``, you can always recover your work from there, but if you
want to delete the previous state, after checking the new one is coherent, you
need to delete these refs otherwise the original object will not be garbage
collected.

.. _backup:

If your repository is on an external hosting service import it
with::

  $ git clone url_of_repository path_of_copy
  $ cd path_of_copy

Even with a local repository, to make experiments without the trouble to
recovering from ``refs/original`` you should get a copy of your
repository with::

     $ git clone path_of_origin path_of_copy
     $ cd path_of_copy
     $ git branch --unset-upstream
     $ git reset --hard

.. index::
   remove from repository

Removing an object or a directory
---------------------------------

Once again this can better be done by now with `git-filter-repo`_ or
`BFG repo cleaner`_. The use of BFG repo cleaner is
:ref:`outlined below <BFG_use>`, the section on filter-repo is still to write.

If you use ``filter-branch`` you can use the argument ``--tree-filter`` or
``--index-filter`` as the second one does not check out the tree, it is a lot
quicker.

When filtering branches you may remove all the changes introduced by
some commit and ends up with empty commit. Some of these emty commits
are useful because they have many parents, i.e. they record a merge.

To avoid such situation you can use ``--prune-empty`` (when you don't
need the incompatible ``--commit-filter``).

Your command will be::

  $ git filter-branch --prune-empty --index-filter  \
      'git rm --cached --ignore-unmatch badfile' HEAD

Here the ``badfile`` is the path of the bad file from the git root,
``git rm`` command has the option ``--cached`` since we are
working on the index and ``--ignore-unmatch`` because the file can be
absent in the index for some commits, like those anterior to the first
occurrence of the file.

If you rather want to delete a full directory content, you will add
the ``-r`` option to make the remove recursive.::

  $ git filter-branch --prune-empty --index-filter  \
      'git rm -r --cached --ignore-unmatch baddir' HEAD

If your object or directory is in many branch, cleaning HEAD will not
get read of it, you should in this case clean all refs and filter all
tags with::

  $ git filter-branch --prune-empty --index-filter  \
      'git rm  --cached --ignore-unmatch badfile' \
      --tag-name-filter cat -- --all

If your unwanted blob has changed name along the history, it will
still be kept with the olders name. You can find the old names
with::

  $ git log --name-only --follow --all -- badfile

And repeat the previous *filter-branch* with these names.

After that your history no longer contains a reference to ``badfile``
but all the ``refs/original/branch`` and the reflog still do. You have
to options, if you have no backup you should do::

      $ git clone file:///path/to/cleanrepo

It is quick since done with hardlinks and the clone will not have the
removed objects.

If you have yet done a backup as proposed :ref:`above <backup>`
you can clean  before repacking.

After a filter-branch, git keep *original* refs. It prevents the
previously referenced object to become loose and be cleaned by garbage
collection. If you want to get rid of them you delete these refs. But
if you want to keep them longer, you should better rename them
to prevent them to be overrode by some next operation (even if  you can
also control the original namespace with ``--original`` option).
::

    $ git for-each-ref --format='%(refname)' refs/original | \
        xargs -n 1 git update-ref -d


Then clean your logs::

  $ git reflog expire --expire=now --all

And you garbage collect all unreferenced objects with::

  $ git gc --prune=now

*More details in the section* :ref:`garbage collection <garbage_collection>`.

Then you can overwite your origin with::

  $ git push origin --force --all
  $ git push origin --force --tags

And warn the other users of the repo, taht if they have no topic
branch they should::

  $ git pull --force origin

Or if they have topics branches, fetch and rebase on the top of the
origin/master.

*Note: Some collaborative hosted repositories will not let you*
``push --force``
*; you will have to delete the reposiotory an push a new one.*

Is my repo containing secrets?
------------------------------

There are at least some utilities to find secret leaks in a git
repository.

- `gittyleaks <https://github.com/kootenpv/gittyleaks>`_ in python.
- `gitleaks <https://github.com/zricethezav/gitleaks>`_ in go
- `repo-scraper <https://github.com/dssg/repo-scraper>`_ in python
   not updated since 2015

::

  $ git  log --unified --all  -S pass1 -S pass2 -s 'password='


  $ git  log --unified --all --reflogs -S pass1 -S pass2 -S 'password='

  $ git  log --unified --exclude='refs/wip'--exclude='refs/remotes' \
  --all -S pass1 -S pass2 -S 'password='

  $ git  log --unified --branches -S pass1 -S pass2 -s 'password='

.. _BFG_use:

Using BFG Repo-Cleaner
----------------------

It is a simpler alternative to *filter-branch*, but more limited than
*git-filter-repo*, you can see a `comparison on the git-filter-repo Readme
<https://github.com/newren/git-filter-repo/blob/main/README.md#bfg-repo-cleaner>`.

`BFG repo cleaner`_ is a scala program, so you can run it if you have the Java
Runtime Environment (Java 7 or above), you don't need scala as the scala
dependencies are packed in a jar archive. It is a lot faster (10 - 720x) than
``git-filter-branch``.

Using it is as simple as::

  $ java -jar bfg.jar --delete-files badfile

BFG can also be used to filter your text blobs and remove
sensitive data.

First you can specify the match expressions in a text file.  Each line is a
match expression, by default, each expression is treated as a literal, but
``regex:` and ``glob:`` prefixes are supported. The default replacement is
``***REMOVED***`` but you can give an alternative replacement with ``==>``.

So your ``secret.txt`` file look like::

  xf54yzt
  glob:key=*
  regex:password=\w+==>password=YOURPASSWORD

Then use bfg with::

  $ bfg --replace-text secret.txt repo.git


.. other refs

    [[http://stackoverflow.com/questions/359424/detach-subdirectory-into-separate-git-repository][stackoverflow - detach a subdirectory]] is now
    [[https://stackoverflow.com/questions/359424/detach-move-subdirectory-into-separate-git-repository//17864475][solved with git subtree]]

..  _git-filter-repo:  https://github.com/newren/git-filter-repo/
..  _BFG repo cleaner: https://rtyley.github.io/bfg-repo-cleaner/
