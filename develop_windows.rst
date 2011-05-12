#####################
Developing on windows
#####################

This is a sketch of the steps that I (MB) went through setting up a
semi-standard development environment for python_ packaging on windows.

************
My system(s)
************

* A standard XP installation booted directly into (My Computer -> Properties)
  XP version 2002, service pack 3.
* An XP running via Parallels_ on my Macbook Air with version 2002 SP3 also.

***********
Basic setup
***********

* Install windows powershell_ . I find this a lot more convenient than the
  standard windows ``cmd`` shell; it has good command and filename completion,
  ``ls`` for directory listing, a less idiosyncratic ``cd``, and a scripting
  language that is a lot less painful than ``cmd`` bat scripting. I believe I've
  used version 1 and version 2 (``$Host.version``` is ``1 0 0 0`` on my standard
  XP installation. Don't forget to enable `quickedit mode
  <http://support.microsoft.com/kb/282301>`_ for much nicer right click copy and
  paste.
* Install msysgit_. In the installation, set ``git`` to be on the command path,
  but not the git bash utilities.  I set the repo to have LF endings, but the
  checked out files to have system endings. You'll see this setting offered in
  the installation GUI - it sets ``core.autocrlf=input`` in your global git
  config.
* Install putty_.  Well, in fact, install all the putty utilities via the
  windows installer - including *plink* and *pageant*.  These are utilities we
  need for handling ssh authentication for git and other tools.
* Get necessary ssh keys via sftp (installed by Putty installer)
* Run *PUTTYgen* (installed by the Putty installer) to import the Unix ssh key
  into putty ``.ppk`` format.  Save ``.ppk`` in ``$HOME/.ssh``.
* Start *pageant* (installed by putty installer).  Add ``.ppk`` key file.
* Check we can get authenticated e.g. into github with ``"c:\Program
  files\Putty\plink git@github.com"``.
* Set a couple of environment variables from powershell - see
  `powershell environment variables`_.  First set ``$GIT_SSH`` to pick up
  *plink* as the ssh executable::

    [Environment]::SetEnvironmentVariable("GIT_SSH", "C:\Program files\Putty\plink.exe", "User")

  Next we make sure that ``$HOME`` is set for safety.  For example, setuptools_
  appears to need it set correctly - see the `example pypi`_ page for the
  assertion at least::

    [Environment]::SetEnvironmentVariable("HOME", $env:USERPROFILE, "User")

  Restart powershell after doing this to pick up the ``GIT_SSH`` environment
  variable in particular.  We should now be able to do something like ``git
  clone git@github.com:my-name/my-repo.git`` without being asked our password
  (pageant handles this).  The first time you use this combination with a
  particular host you may get an error like this::

    The server's host key is not cached in the registry. You
    have no guarantee that the server is the computer you
    think it is.

  If so, run *plink* manually to cache the key::

    & 'C:\Program Files\PuTTY\plink.exe' git@github.com

  and press ``y`` when asked.
* Install editor.  I use vim_.  You may also want to install command-t - see
  [#nasty-install]_.
* Install python_ - the current version.  I need this for the scripts installing
  the personal configuration below.  For convenience in running other things,
  you might want to make sure python is on the path.
* Set up personal configuration.  For me this is something like::

    cd c:\
    mkdir code
    cd code
    mkdir dev_trees
    cd dev_trees
    git clone git@github.com:matthew-brett/myconfig.git
    git clone git@github.com:matthew-brett/myvim.git
    cd myconfig
    c:\Python26\python make.py dotfiles
    cd ..\myvim
    c:\Python26\python make.py vimfiles

  For using ssh inside the git bash shell, I like to add the keychain-like
  script, by including this in my ``~/.bashrc`` file::

    # Source personal definitions
    if [ -f "$HOME/.bash_keychain_lite" ]; then
        . "$HOME/.bash_keychain_lite"
    fi

  where the ``.bash_keychain_lite`` file arises from the ``make.py dotfiles``
  command above.

.. _win-compile-tools:

********************************
Windows compiler and build tools
********************************

* Download and install the mingw_ windows compiler.  I used the
  ``mingw-get-inst`` automated installation route.  Select the options giving
  you c++, fortran, and the msys build environment.  I didn't directly add these
  to the path, but made a script ``c:\Mingw\mingwvar.ps1``::

    # convenience script to add mingw to path
    echo "Adding mingw to PATH..."
    $mingw = [System.IO.Path]::GetDirectoryName($MyInvocation.MyCommand.Definition)
    $env:path = "$mingw\bin;$mingw\msys\1.0\bin;$env:path"

  Then in powershell - ``source c:\Mingw\mingwvar.ps1`` to add the msys and
  mingw tools to the path.

* Using mingw, you might get this kind of error::

    error: Setup script exited with error: Unable to find vcvarsall.bat

  Of course you've already tried the standard solution to this, of the form::

    python setup.py build --compiler=mingw32

  If that doesn't work, you might have hit a `mingw distutils bug`_.  One
  suggested fix is to make a ``distutils.cfg`` file in your Python distutils
  directory (e.g.  ``C:\Python26\Lib\distutils``) with the following content::

    [build]
    compiler=mingw32

.. _powershell environment variables: http://technet.microsoft.com/en-us/library/ff730964.aspx
.. _mingw distutils bug: http://bugs.python.org/issue2698

.. rubric:: Footnotes

.. [#nasty-install] command-t_ - as of January 2011, required vim 7.2.
    See the `command-t README`_.  We need vim 7.2 because of ruby
    incompatibility problems.  vim 7.2 uses python 2.4, so you might want that
    installed (not necessarily as default).  We need ruby 1.8.7, and the ruby
    devkit.  Set the install option to add ruby to the path when running the
    ruby installer.  Unpack the devkit to ``c:\devkit``.  Start powershell and
    source the devkit variables with ``c:\devkit\devkitvar.ps1``.  Then::

        cd ~\vimfiles\bundle\command-t\ruby\command-t
        ruby extconf.rb
        make
