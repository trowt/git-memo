..  _filter_branch:

.. index::
    pair: filter; branch
    single: git; filter-branch

Filter branch
=============

.. warning::

   Note that the ``filter-branch`` is officially not recommended. Recent
   versions of the ``filter-branch`` manual contain the following warning:

      ``git filter-branch`` has a plethora of pitfalls that can produce
      non-obvious manglings of the intended history rewrite (and can leave you
      with little time to investigate such problems since it has such abysmal
      performance).  These safety and performance issues cannot be backward
      compatibly fixed and as such, its use is not recommended. Please use an
      alternative history filtering tool such as ``git filter-repo``. If you
      still need to use ``git filter-branch``, please carefully read SAFETY
      (and PERFORMANCE) to learn about the land mines of ``filter-branch``, and
      then vigilantly avoid as many of the hazards listed there as reasonably
      possible.

This chapter describe some uses of ``git filter-branch`` before using
it you should be aware that this command is destructive, and even the
untouched commits end up with different object names so your new
branch is separate from the original one. If ever you repository was
shared anyone downstream  is forced to manually fix their history,
by rebasing all their topic branches over the new HEAD.
More details in
:gitdoc:`git-rebase(1) - recovering from upstream rebase
<git-rebase.html#_recovering_from_upstream_rebase>`

When filtering branches the original refs, are stored in the namespace
``refs/original/``, you can always recover your work from there, but if
you want to delete the previous state, after checking the new one is
coherent, you need to delete these refs otherwise the original object
will not be garbage collected.

.. _backup:

If you want to make experiments without the trouble to recovering from
``refs/original`` you should get a  copy of your repository
with::

     git clone path_of_origin path_of_copy
     cd path_of_copy
     git branch --unset-upstream
     git reset --hard





References
----------

-   The main reference is :gitdoc:`git documentation: filter-branch
    <git-filter-branch.html>`
-   It is introduced in S. Chacon Pro-Git `Rewriting History chapter
    <http://git-scm.com/book/ch6-4.html#The-Nuclear-Option:-filter-branch>`_
    and `in Maintenance and Data Recovery - removing objects
    <http://git-scm.com/book/ch9-7.html#Removing-Objects>`_.


Removing an object or a directory
---------------------------------

This can be done with ``--tree-filter`` or ``-index-filter`` as the
second one does not check out the tree, it is a lot quicker.

When filtering branches you may remove all the changes introduced by
some commit and ends up with empty commit. Some of these emty commits
are useful because they have many parents, i.e. they record a merge.

To avoid such situation you can use ``--prune-empty`` (but it is
incompatible with ``--commit-filter``.

Your command will be::

  git filter-branch --prune-empty --index-filter  \
      'git rm --cached --ignore-unmatch badfile' HEAD

Here the ``git rm`` command has the option ``--cached`` since we are
working on the index and ``--ignore-unmatch`` because the file can be
absent in the index for some commits, like those anterior to the first
occurrence of the file.

If you rather want to delete a full directory content, you will add
the ``-r`` option to make the remove recursive.::

  git filter-branch --prune-empty --index-filter  \
      'git rm -r --cached --ignore-unmatch baddir' HEAD

If your object or directory is in many branch, cleaning HEAD will not
get read of it, you should in this case clean all refs and filter all
tags with::

  git filter-branch --prune-empty --index-filter  \
      'git rm  --cached --ignore-unmatch badfile' \
      -tag-name-filter cat -- --all

If your unwanted blob has changed name along the history, it will
still be kept with the olders name, but if you take care to find them
with::

  git log --name-only --follow --all -- badfile

After that your history no longer contains a reference to ``badfile``
but all the ``refs/original/branch`` and the reflog still do. You have
to options, if you have no backup you should do::

      git clone file:///path/to/cleanrepo

It is quick since done with hardlinks and the clone will not have the
removed objects.

If you have yet done a backup as proposed :ref:`above <backup>`
you can clean  before repacking.
After a filter-branch git keep *original* refs, that prevent the
previously referenced object to become loose and be cleaned by garbage
collection. If you want to get rid of them you delete these refs, on
the other side if you want to keep them longer, you better rename them
to prevent them to be overrode by some next operation (even if  you can
also control the original namespace with ``--original`` option).
::

    git for-each-ref --format='%(refname)' refs/original | \
	xargs -n 1 git update-ref -d


Then your logs::

  git reflog expire --expire=now --all

And you garbage collect all unreferenced objects with::

  git gc --prune=now

*More details in the section* :ref:`garbage collection <garbage_collection>`.

*Note:  Many collaborative hosted repositories like GitHub,
BitBucket and others, will not let you push back your deletes, so if
you really want to be sure nobody can get your old file, you will have
to delete these repos an push new ones.*

.. other refs

    [[http://stackoverflow.com/questions/359424/detach-subdirectory-into-separate-git-repository][stackoverflow - detach a subdirectory]]
