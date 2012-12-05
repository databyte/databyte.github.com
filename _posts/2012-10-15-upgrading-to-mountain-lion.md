---
layout: post
title: Upgrading to Mountain Lion
---

So I finally got around to upgrading to Mountain Lion.

#### Required Steps

My first stop was to follow Thoughtbot's
[The Hitchhiker's Guide to Riding a Mountain Lion](http://robots.thoughtbot.com/post/27985816073/the-hitchhikers-guide-to-riding-a-mountain-lion).

* Get Xcode and Command Line Tools installed
* Fix homebrew permissions with `sudo chown -R $USER /usr/local`
* Update homebrew with `brew update`
* Install gcc 4.2 to install older rubies with `brew tap homebrew/dupes;
  brew install autoconf automake apple-gcc42`
* Install X11 support with [XQuartz](http://xquartz.macosforge.org/landing)

#### TL;DR for installing Ruby 1.8.7

    $ export CPPFLAGS=-I/opt/X11/include
    $ CC=/usr/local/bin/gcc-4.2 rvm install 1.8.7

#### My little story

Initially I didn't want to install X11 since I didn't think I needed it.

Then I tried to use `vim` (not MacVim, I just wanted to edit a file
quickly) and received:

    $ vim
    Vim: Caught deadly signal SEGV
    Vim: Finished.
    [1]    42573 segmentation fault  vim

What version of vim?

    $ vim --version
    VIM - Vi IMproved 7.3 (2010 Aug 15, compiled Jun 20 2012 13:16:02)
    Compiled by root@apple.com
    /snip/

It seems that if you `brew install vim`, it still breaks:

    $ /usr/local/bin/vim --version
    VIM - Vi IMproved 7.3 (2010 Aug 15, compiled Oct 15 2012 18:02:59)
    MacOS X (unix) version
    Included patches: 1-687
    Compiled by databyte@nyx

Turns out that I need ruby-1.8.7 due to a compatibility in
[Janus](https://github.com/carlhuda/janus), which was outdated. The
quick fix was upgrading Janus by running `rake` within `~/.vim` but I
might as well compile ruby-1.8.7 since I bothered to install gcc 4.2.

Let's try to install ruby-1.8.7 with rvm:

    $ rvm install ruby-1.8.7-p370
    No binary rubies available for: osx/10.8/x86_64/ruby-1.8.7-p370.
    Continuing with compilation. Please read 'rvm mount' to get more information on binary rubies.
    Installing Ruby from source to: /Users/databyte/.rvm/rubies/ruby-1.8.7-p370, this may take a while depending on your cpu(s)...
    ruby-1.8.7-p370 - #downloading ruby-1.8.7-p370, this may take a while depending on your connection...
    ruby-1.8.7-p370 - #extracting ruby-1.8.7-p370 to /Users/databyte/.rvm/src/ruby-1.8.7-p370
    ruby-1.8.7-p370 - #extracted to /Users/databyte/.rvm/src/ruby-1.8.7-p370
    Applying patch /Users/databyte/.rvm/patches/ruby/1.8.7/stdout-rouge-fix.patch
    Applying patch /Users/databyte/.rvm/patches/ruby/1.8.7/no_sslv2.diff
    ruby-1.8.7-p370 - #configuring
    ruby-1.8.7-p370 - #compiling
    Error running 'make', please read /Users/databyte/.rvm/log/ruby-1.8.7-p370/make.log
    There has been an error while running make. Halting the installation.
    Please be aware that you just installed a ruby that requires        2 patches just to be compiled on up to date linux system.
    This may have known and unaccounted for security vulnerabilities.
    Please consider upgrading to Ruby 1.9.3-286 which will have all of the latest security patches.

Turns out in the logs that:

    /usr/include/tk.h:78:23: error: X11/Xlib.h: No such file or directory

Hence the need for XQuartz. Now let's install XQuartz and then reinstall ruby
but also explicity set X11's include path:

    $ export CPPFLAGS=-I/opt/X11/include
    $ CC=/usr/local/bin/gcc-4.2 rvm reinstall 1.8.7
    Removing /Users/databyte/.rvm/src/ruby-1.8.7-p370...
    /Users/databyte/.rvm/rubies/ruby-1.8.7-p370 has already been removed.
    No binary rubies available for: osx/10.8/x86_64/ruby-1.8.7-p370.
    Continuing with compilation. Please read 'rvm mount' to get more information on binary rubies.
    Installing Ruby from source to: /Users/databyte/.rvm/rubies/ruby-1.8.7-p370, this may take a while depending on your cpu(s)...
    ruby-1.8.7-p370 - #downloading ruby-1.8.7-p370, this may take a while depending on your connection...
    ruby-1.8.7-p370 - #extracting ruby-1.8.7-p370 to /Users/databyte/.rvm/src/ruby-1.8.7-p370
    ruby-1.8.7-p370 - #extracted to /Users/databyte/.rvm/src/ruby-1.8.7-p370
    Applying patch /Users/databyte/.rvm/patches/ruby/1.8.7/stdout-rouge-fix.patch
    Applying patch /Users/databyte/.rvm/patches/ruby/1.8.7/no_sslv2.diff
    ruby-1.8.7-p370 - #configuring
    ruby-1.8.7-p370 - #compiling
    ruby-1.8.7-p370 - #installing
    Removing old Rubygems files...
    Installing rubygems-1.8.24 for ruby-1.8.7-p370 ...
    Installation of rubygems completed successfully.
    Saving wrappers to '/Users/databyte/.rvm/bin'.
    ruby-1.8.7-p370 - #adjusting #shebangs for (gem irb erb ri rdoc testrb rake).
    ruby-1.8.7-p370 - #importing default gemsets (/Users/databyte/.rvm/gemsets/)
    Install of ruby-1.8.7-p370 - #complete
    Please be aware that you just installed a ruby that requires        2 patches just to be compiled on up to date linux system.
    This may have known and unaccounted for security vulnerabilities.
    Please consider upgrading to Ruby 1.9.3-286 which will have all of the latest security patches.
    Making gemset ruby-1.8.7-p370 pristine.
    Making gemset ruby-1.8.7-p370@global pristine.

Well that works and now I have ruby 1.8.7. Let's try out vim again:

    $ rvm use 1.8.7
    $ /usr/local/bin/vim

Yep, that works too!

    $ vim --version
    VIM - Vi IMproved 7.3 (2010 Aug 15, compiled Oct 15 2012 18:08:38)
    MacOS X (unix) version
    Included patches: 1-687
    Compiled by databyte@
    Huge version without GUI.  Features included (+) or not (-):
    +arabic +autocmd -balloon_eval -browse ++builtin_terms +byte_offset +cindent
    -clientserver +clipboard +cmdline_compl +cmdline_hist +cmdline_info +comments
    +conceal +cryptv +cscope +cursorbind +cursorshape +dialog_con +diff +digraphs
    -dnd -ebcdic +emacs_tags +eval +ex_extra +extra_search +farsi +file_in_path
    +find_in_path +float +folding -footer +fork() -gettext -hangul_input +iconv
    +insert_expand +jumplist +keymap +langmap +libcall +linebreak +lispindent
    +listcmds +localmap -lua +menu +mksession +modify_fname +mouse -mouseshape
    +mouse_dec -mouse_gpm -mouse_jsbterm +mouse_netterm -mouse_sysmouse
    +mouse_xterm +mouse_urxvt +mouse_sgr +multi_byte +multi_lang -mzscheme
    +netbeans_intg +path_extra -perl +persistent_undo +postscript +printer +profile
     +python -python3 +quickfix +reltime +rightleft +ruby +scrollbind +signs
    +smartindent -sniff +startuptime +statusline -sun_workshop +syntax +tag_binary
    +tag_old_static -tag_any_white -tcl +terminfo +termresponse +textobjects +title
     -toolbar +user_commands +vertsplit +virtualedit +visual +visualextra +viminfo
    +vreplace +wildignore +wildmenu +windows +writebackup -X11 -xfontset -xim -xsmp
     -xterm_clipboard -xterm_save
       system vimrc file: "$VIM/vimrc"
         user vimrc file: "$HOME/.vimrc"
          user exrc file: "$HOME/.exrc"
      fall-back for $VIM: "/usr/local/share/vim"
    Compilation: cc -c -I. -Iproto -DHAVE_CONFIG_H   -DMACOS_X_UNIX -no-cpp-precomp  -g -O2 -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=1
    Linking: cc   -L.    -L/usr/local/lib -o vim       -lm  -lncurses -liconv -framework Cocoa     -framework Python   -lruby -lobjc

