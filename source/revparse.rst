Objects names
=============

.. _commit_names:

Naming commits
--------------

.. index::
   symbolic name
   symbolic reference
   single: git;rev-parse

See :gitdoc:`git-rev-parse <git-rev-parse.html>`,
:gitdoc:`gitrevisions(7) <gitrevisions.html>`,
:gitdoc:`git-name-rev(1) <git-name-rev.html>`,
:gitdoc:`git-describe(1) <git-describe.html>`,
:gitdoc:`git-reflog(1) <git-reflog.html>`

The ref name can be used in most git commands. As an example we show
here the use with :gitdoc:`git-name-rev <git-name-rev.html>` and
:gitdoc:`git-rev-parse <git-rev-parse.html>`, but you
can use it wherever a revision is needed.


-   To inspect your repository (and in scripts)

    ::

        $ pwd
        /home/marc/bash/lib
        $ git rev-parse  --is-inside-git-dir
        false
        $ git rev-parse  --is-inside-work-tree
        true
        $ git rev-parse --git-dir
        /shared/home/marc/bash/.git
        $ git rev-parse --show-cdup
        ../
        $ git rev-parse --show-prefix
        lib/

.. index::
   single: git; symbolic-ref
   single: git; name-ref

-   translating symbolic <-> sha
    ::

        $ git rev-parse --symbolic-full-name HEAD
        refs/heads/master
        $ git symbolic-ref HEAD
        refs/heads/master
        $ git name-rev --name-only HEAD
        master
        $ git rev-parse  HEAD~3
        25f4b1d58e20f2026a36d80073654f52b055537b
        $ git name-rev --name-only 25f4b1d58e20
        master~3

-  Where is this commit sha?

   Sometime you have a commit given by a revision sha number, say
   `1fc1148f`, like the one you find in a
   cherry-pick crossref. You may want to know where it is in your
   repository.

   A simple :gitdoc:`describe <git-describe.html>` will only succeed if
   it is referenced by a tag.  but you can use *--all* to match also
   all branches, and *--contains* to find not only the branch but all
   the contained refs.
   ::

       $ git describe --all --contains 1fc1148f86
       remotes/origin/distrib~11

   :gitdoc:`name-rev <git-name-rev.html>` will also give the same
   symbolic ref.
   ::

       $ git name-rev --name-only 1fc1148f86
       remotes/origin/distrib~11

   We know that ``1fc1148f86`` is the eleventh commit from the
   ``remotes/origin/distrib`` branch, but I prefer to name it relative
   to the local `distrib` branch.

   We can use the `--refs` option yo limit the refs name, but using:
   ::

     $ git name-rev --name-only --refs=distrib 1fc1148f867
     origin/distrib~11

   give the same answer than previously because ``origin-distrib``  is
   also matched.

   To be more specific we use:
   ::

       $ git name-rev --name-only --refs=heads/distrib 1fc1148f867
       distrib~12

   If we want to see both the message and a symbolic ref we can do:
   ::

       $ git log -1 distrib~12 | git name-rev --stdin
       commit 1fc1148f867ee644f9c039fd3614ae5c48171276 (remotes/origin/distrib~11)
       Author: .....
       ....

.. index::
   pair:git;describe

-   version/most recent tag

    ::

        $ git describe HEAD
        init-1.0-29-gcb97cd9
        $ git name-rev --name-only cb97cd9
        master
        $ git describe HEAD~14
        init-1.0-15-g84aeca4
        $ git name-rev --name-only 84aeca4
        master~14
        $ git describe HEAD~29
        init-1.0
        $ git describe --long HEAD~29
        init-1.0-0-ge23c217


.. index::
   pair:git;name-rev

.. _reflog_history:

-   past tips of branches

    We use the reflog, be careful that the reflog is local to your
    repository, and is pruned by ``git reflog expire`` or by ``git gc``
    ``HEAD@{25}`` is the 25th older head of branch, this is not always
    the same than ``HEAD~25`` which is the 25th ancestor of the
    actual head.
    ::

        $ git name-rev HEAD@{25}
        HEAD@{25} b3distrib~11
        $ git rev-parse HEAD@{25}
        2518dd006de12f8357e9694bf51a27bbd5bb5c7a
        $ git rev-parse HEAD~11
        2518dd006de12f8357e9694bf51a27bbd5bb5c7a
        $ git name-rev 2518dd0
        2518dd0 b3distrib~11
        $ git rev-parse HEAD@{18}
        0c4c8c0ea9ab54b92a2a6d2fed51d19c50cd3d76
        $ git name-rev HEAD@{18}
        HEAD@{18} undefined
        $ git rev-parse HEAD@{14}~4
        0c4c8c0ea9ab54b92a2a6d2fed51d19c50cd3d76
        $ git rev-parse HEAD@{13}~5
        24c85381f6d7420366e7a5e305c544a44f34fb0f
        git log -1 -g --oneline HEAD@{13}
        a1b9b5c HEAD@{13}: checkout: moving from b3distrib to a1b9b5c

    In the previous example The 13th ancestor from the ``HEAD`` is a
    checkout at the beginning of a rebase so ``HEAD@{14}`` is now
    dangling, and ``HEAD@{18}`` the fourth predecessor (``HEAD@{14}~4``) of
    ``HEAD@{14}`` is unreachable from a ref.

    Nevertheless ``HEAD@{25}`` has been rebased as  ``HEAD~11`` and
    can be reached.

-   Commits at a certain time. We can look at the *local* log for a
    commit made at a specific time. If we want to look at the HEAD
    branch, we can use a ``@{date}``. Exemples

    ::

        $ git rev-parse @{1month}
        97491a2d5d6a668116acff359ed4fd874d800f6f
        $ git rev-parse @{1week}
        a400ee44b01c093c595c79c15b8f049713daeb78
        $ git rev-parse @{2017-09-26}
        97491a2d5d6a668116acff359ed4fd874d800f6f

    Note that this syntax look only at the local log, it does not mean that
    this commit is still in the history of the branch.

    ::

        $ git name-rev --name-only @{1week}
        master~5
        $ git name-rev --name-only @{3weeks}
        undefined

    .. Comment

        The following does not work as expected, should check why.

        If I want to know the actual log of the present HEAD, 3 weeks
        ago I will not use ``@{3weeks}`` but

        ::

            $ git log --until='3 weeks'

    If we want to explore the log of another branch we use

    ::

        $ git rev-parse dev@{1hour}

..  index::
    pair object; sha
    single: file; sha
    single: git;ls-files
    single: git; ls-tree
    hash
    single:git; hash-object


Finding the sha of a file
-------------------------

Refs:
    :gitdoc:`git ls-files(1) <git-ls-tree.html>`,
    :gitdoc:`git ls-tree(1) <git-ls-tree.html>`,
    :gitdoc:`git-rev-parse(1) <git-rev-parse.html>`,
    :gitdoc:`gitrevisions(7) <gitrevisions.html>`,
    :gitdoc:`git hash-object(1) <git-hash-object.html>`.

    :progit:`Pro Git: Git Objects<Git-Internals-Git-Objects>`,
    `Discussion by Linus Torvald
    <http://article.gmane.org/gmane.comp.version-control.git/44849>`_

To show the blog sha associated with a file **in the index**:

::

    $ git ls-files --stage somefile
    100644 a8ca07da52ba219e2c76685b7e59b34da435a007 0	somefile

This is **not** the *sha1 sum*
of the raw content, but you can get it
from any file *even unknown in your repository* with::

    $ git hash-object somefile
    a8ca07da52ba219e2c76685b7e59b34da435a007
    $ cat somefile | git hash-object --stdin
    a8ca07da52ba219e2c76685b7e59b34da435a007

The sha is derived from the content, and the size of the file, you can
get it without using git from the `sha1sum
<http://manpages.debian.org/cgi-bin/man.cgi?query=sha1sum>`_
command with::

    $ (/usr/bin/stat  --printf "blob %s\0" somefile; cat somefile) | \
      sha1sum
    a8ca07da52ba219e2c76685b7e59b34da435a007

While :gitdoc:`git ls-file <git-ls-files.html>` use by default the cached
content, by using plumbing commands, you can also look at any object.

To show the blog sha of the
object associated with a relative path in the *HEAD*::

    $ git ls-tree HEAD <path>

You can also use path starting from the git worktree directory.
If the root of your are in a directory *subdir* you get the same
result with::

    $ git ls-tree HEAD somefile
    100644 blob 1a8bedab89a0689886cad63812fca9918d194a98	somefile
    $ git ls-tree HEAD :somefile
    100644 blob 1a8bedab89a0689886cad63812fca9918d194a98	somefile
    $ git ls-tree HEAD :./somefile
    100644 blob 1a8bedab89a0689886cad63812fca9918d194a98	somefile
    git ls-tree HEAD :/subdir/file #note initial slash
    100644 blob 1a8bedab89a0689886cad63812fca9918d194a98	somefile

you can also use :gitdoc:`git rev-parse <git-rev-parse.html>` with::

    $ git rev-parse HEAD:subdir/somefile # no leading slash
    1a8bedab89a0689886cad63812fca9918d194a98
    $ git rev-parse HEAD:./somefile
    1a8bedab89a0689886cad63812fca9918d194a98
    $ git rev-parse :./somefile # index cached content
    a8ca07da52ba219e2c76685b7e59b34da435a007
    $ git rev-parse :0:./somefile
    a8ca07da52ba219e2c76685b7e59b34da435a007
    $ git hash-object somefile # the unregisterd worktree version
    67a21c581328157099e8eac97b063cff2fb1a807  somefile


Finding the top level directory
-------------------------------

.. index::
   single: git;rev-parse

Ref: :gitdoc:`git-rev-parse(1) <git-rev-parse.html>`

To show the absolute path of the top-level directory.:
::

    $ git rev-parse --show-toplevel

To show the *relative* path of the top-level repository::

    $ git rev-parse --show-cdup

or to show the path of the current directory relative to the
top-level::

    $ git rev-parse --show-prefix

I use it to have a default message showing paths relative to top-level
with::

    $ git commit :/$(git rev-parse --show-prefix)<relative-name>


To show the git directory:
::

    $ git rev-parse --git-dir

If ``$GIT_DIR`` is defined it is  returned otherwise when we are in
Git directory return the ``.git`` directory, if not exit with nonzero
status after printing an error message.

To know if you are in a work-tree::

    $ git rev-parse --is-inside-work-tree

Note also that an alias expansion  prefixed with an exclamation point
will be executed from the top-level directory of a repository
i.e. from ``git rev-parse --show-toplevel``.
