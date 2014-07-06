# Extended session management for Vim

The vim-session plug-in improves upon [Vim](http://www.vim.org/)'s built-in [:mksession][mksession] command by enabling you to easily and (if you want) automatically persist and restore your Vim editing sessions. It works by generating a [Vim script](http://vimdoc.sourceforge.net/htmldoc/usr_41.html#script) that restores your current settings and the arrangement of tab pages and/or split windows and the files they contain.

To persist your current editing session you can execute the `:SaveSession` command. If you don't provide a name for the session 'default' is used (you can change this name with an option). You're free to use whatever characters you like in session names. When you want to restore your session simply execute `:OpenSession`. Again the name 'default' is used if you don't provide one. When a session is active, has been changed and you quit Vim you'll be prompted whether you want to save the open session before quitting Vim:

![Screenshot of auto-save prompt](http://peterodding.com/code/vim/session/autosave.png)

If you want, the plug-in can also automatically save your session every few minutes (see the `g:session_autosave_periodic` option). When you start Vim without editing any files and the default session exists, you'll be prompted whether you want to restore the default session:

![Screenshot of auto-open prompt](http://peterodding.com/code/vim/session/autoopen.png)

When you start Vim with a custom [server name](http://vimdoc.sourceforge.net/htmldoc/remote.html#--servername) that matches one of the existing session names then the matching session will be automatically restored. For example I use several sessions to quickly edit my Vim plug-ins:

    $ gvim --servername easytags-plugin
    $ gvim --servername session-plugin
    $ # etc.

The session scripts created by this plug-in are stored in the directory `~/.vim/sessions` (on UNIX) or `~\vimfiles\sessions` (on Windows) but you can change the location by setting `g:session_directory`. If you're curious what the session scripts generated by vim-session look like see the [sample below](http://peterodding.com/code/vim/session/#sample_session_script).

If you're still getting to know the plug-in, the "Sessions" menu may help: It contains menu items for most commands defined by the plug-in.

## Installation

*Please note that the vim-session plug-in requires my vim-misc plug-in which is separately distributed.*

Unzip the most recent ZIP archives of the [vim-session] [download-session] and [vim-misc] [download-misc] plug-ins inside your Vim profile directory (usually this is `~/.vim` on UNIX and `%USERPROFILE%\vimfiles` on Windows), restart Vim and execute the command `:helptags ~/.vim/doc` (use `:helptags ~\vimfiles\doc` instead on Windows). To get started execute `:Note` or `:edit note:`, this will start a new note that contains instructions on how to continue from there (and how to use the plug-in in general).

If you prefer you can also use [Pathogen] [pathogen], [Vundle] [vundle] or a similar tool to install & update the [vim-session] [github-session] and [vim-misc] [github-misc] plug-ins using a local clone of the git repository.

After you've installed the plug-in and restarted Vim, the following commands will be available to you:

## Commands

Note that environment variables inside command arguments are expanded by the plug-in.

### The `:SaveSession` command

This command saves your current editing session just like Vim's built-in [:mksession][mksession] command does. The difference is that you don't pass a full pathname as argument but just a name, any name really. Press `<Tab>` to get completion of existing session names. If you don't provide an argument the default session name is used, unless an existing session is open in which case the name of that session will be used.

If the session you're trying to save is already active in another Vim instance you'll get a warning and nothing happens. You can use a bang (!) as in `:SaveSession! ...` to ignore the warning and save the session anyway.

Your session script will be saved in the directory pointed to by the `g:session_directory` option.

### The `:OpenSession` command

This command is basically [:source][source] in disguise, but it supports tab completion of session names and it executes `:CloseSession` before opening the session. When you don't provide a session name and only a single session exists then that session is opened, otherwise the plug-in will ask you to select one from a list:

    Please select the session to restore:

     1. vim-profile
     2. session-plugin
     3. etc.

    Type number and <Enter> or click with mouse (empty cancels):

If the session you're trying to open is already active in another Vim instance you'll get a warning and nothing happens. You can use a bang (!) as in `:OpenSession! ...` to ignore the warning and open the session anyway.

Note also that when you use a bang (!) right after the command name existing tab pages and windows are closed, discarding any changes in the files you were editing!

### The `:RestartVim` command

This command saves your current editing session, restarts Vim and restores your editing session. This can come in handy when you're debugging Vim scripts which can't be easily/safely [reloaded using a more lightweight approach](http://peterodding.com/code/vim/reload/). It should work fine on Windows and UNIX alike but because of technical limitations it only works in graphical Vim.

Any commands following the `:RestartVim` command are intercepted and executed after Vim is restarted and your session has been restored. This makes it easy to perform manual tests which involve restarting Vim, e.g. `:RestartVim | edit /path/to/file | call MyTest()`.

### The `:CloseSession` command

This command closes all but the current tab page and window and then edits a new, empty buffer. If a session is loaded when you execute this command the plug-in will first ask you whether you want to save that session.

Note that when you use a bang (!) right after the command name existing tab pages and windows are closed, discarding any changes in the files you were editing!

### The `:DeleteSession` command

Using this command you can delete any of the sessions created by this plug-in. If the session you are trying to delete is currently active in another Vim instance you'll get a warning and nothing happens. You can use a bang (!) as in `:DeleteSession! ...` to ignore the warning and delete the session anyway.

Note that this command only deletes the session script, it leaves your open tab pages and windows exactly as they were.

### The `:ViewSession` command

Execute this command to view the Vim script generated for a session. This command is useful when you need to review the generated Vim script repeatedly, for example while debugging or modifying the vim-session plug-in.

### Tab scoped sessions

When ['sessionoptions'] [sessionoptions] contains 'tabpages' (this is the default) session scripts will persist and restore all windows in all tab pages. When you remove 'tabpages' from ['sessionoptions'] [sessionoptions] you get a sort of light-weight sessions: They are constrained to a single tab page. Vim's [:mksession] [mksession] command and the vim-session plug-in both fully support this.

You can change ['sessionoptions'] [sessionoptions] in your [vimrc script] [vimrc] but then you can never save a session including tab pages. To decide on the spot whether you want a global or tab scoped session, the vim-session plug-in defines the three commands documented below.

Note that tab scoped sessions are regular session scripts, so when you load a tab scoped session using `:OpenSession` instead of `:OpenTabSession` the vim-session plug-in *assumes* it is a global session and will close all active tab pages before opening the tab scoped session.

#### The `:OpenTabSession` command

Just like `:OpenSession` but applies only to the current tab page.

#### The `:SaveTabSession` command

Just like `:SaveSession` but applies only to the current tab page.

#### The `:AppendTabSession` command

This command opens a new tab page and loads the given tab scoped session in that tab page. You can give this command a count just like [:tabnew] [tabnew].

#### The `:CloseTabSession` command

Just like `:CloseSession` but applies only to the current tab page.

## Options

The following Vim options and plug-in options (global variables) can be used to configure the plug-in to your preferences.

### The `sessionoptions` setting

Because the vim-session plug-in uses Vim's [:mksession][mksession] command you can change how it works by setting ['sessionoptions'][sessionoptions] in your [vimrc script] [vimrc], for example:

    " If you only want to save the current tab page:
    set sessionoptions-=tabpages

    " If you don't want help windows to be restored:
    set sessionoptions-=help

Note that the vim-session plug-in automatically and unconditionally executes the following change just before saving a session:

    " Don't persist options and mappings because it can corrupt sessions.
    set sessionoptions-=options

### The `g:session_directory` option

This option controls the location of your session scripts. Its default value is `~/.vim/sessions` (on UNIX) or `~\vimfiles\sessions` (on Windows). If you don't mind the default you don't have to do anything; the directory will be created for you. Note that a leading `~` is expanded to your current home directory (`$HOME` on UNIX, `%USERPROFILE%` on Windows).

### The `g:session_default_name` option

The name of the default session without directory or filename extension (you'll never guess what the default is).

### The `g:session_default_overwrite` option

If you set this to true (1), every Vim instance without an explicit session loaded will overwrite the default session (the last Vim instance wins).

### The `g:session_extension` option

The filename extension of session scripts. This should include the dot that separates the basename from the extension. Defaults to '.vim'.

### The `g:session_autoload` option

By default this option is set to `'prompt'`. This means that when you start Vim without opening any files and the default session script exists, the session plug-in will ask whether you want to restore your default session. When you set this option to `'yes'` and you start Vim without opening any files the default session will be restored without a prompt. To completely disable automatic loading you can set this option to `'no'`.

### The `g:session_autosave` option

By default this option is set to `'prompt'`. When you've opened a session and you quit Vim, the session plug-in will ask whether you want to save the changes to your session. Set this option to `'yes'` to always automatically save open sessions when you quit Vim. To completely disable automatic saving you can set this option to `'no'`.

### The `g:session_autosave_periodic` option

This option sets the interval in minutes for automatic, periodic saving of active sessions. The default is zero which disables the feature.

Note that when the plug-in automatically saves a session (because you enabled this feature) the plug-in will not prompt for your permission.

### The `g:session_verbose_messages` option

The session load/save prompts are quite verbose by default because they explain how to disable the prompts. If you find the additional explanation distracting you can lower the verbosity by setting this option to 0 (false) in your [vimrc script] [vimrc].

### The `g:session_default_to_last` option

By default this option is set to false (0). When you set this option to true (1) and you start Vim, the session plug-in will open your last used session instead of the default session. Note that the session plug-in will still show you the dialog asking whether you want to restore the last used session. To get rid of the dialog you have to set `g:session_autoload` to `'yes'`.

### The `g:session_persist_colors` option

By default the plug-in will save the color scheme and the ['background' option] [bg] with the session to be reused the next time that session is loaded, this can be disabled by adding the following line to your [vimrc script] [vimrc]:

    :let g:session_persist_colors = 0

### The `g:session_persist_globals` option

The vim-session plug-in uses Vim's [:mksession] [mksession] command but it changes ['sessionoptions'][sessionoptions] so that Vim options and mappings are not persisted. The plug-in does this because persistence of options and mappings can break loading of sessions, in other words it's fragile (in my opinion).

If you want the plug-in to persist specific global variables or options you can add their names to the list `g:session_persist_globals` in your [vimrc script] [vimrc]:

    " Persist the value of the global option 'makeprg'.
    let g:session_persist_globals = ['&makeprg']

Because the [vimrc script] [vimrc] is loaded before the plug-in you have to define the list yourself. To persist multiple values:

    " Persist all options related to :make.
    let g:session_persist_globals = ['&makeprg', '&makeef']

Here's how you persist global variables: (in this case the variables of the session plug-in itself :-)

    " Persist the options of the session plug-in using the session plug-in...
    let g:session_persist_globals = ['&sessionoptions']
    call add(g:session_persist_globals, 'g:session_autoload')
    call add(g:session_persist_globals, 'g:session_autosave')
    call add(g:session_persist_globals, 'g:session_default_to_last')
    call add(g:session_persist_globals, 'g:session_persist_globals')

The example above doesn't persist the `g:session_directory` variable because this variable is used before loading a session script so persisting it inside the session script is pointless.

### The `g:session_restart_environment` option

This option is a list of environment variable names (without the dollar signs) that the `:RestartVim` command will pass on to the new instance of Vim. This option is only useful on UNIX. By default the three environment variables `$TERM`, `$VIM` and `$VIMRUNTIME` are included in this list.

### The `g:session_command_aliases` option

The names of the commands defined by the session plug-in start with the action they perform, followed by the string 'Session'. Some people prefer it the other way around because they find it easier to remember and you can type `:Session<Tab>` to get completion of all available commands (actually this works with the other style as well if you type `:*Session<Tab>` but I digress). If you are one of those people you can enable this option in your [vimrc script] [vimrc] like this:

    :let g:session_command_aliases = 1

When this option is enabled the session plug-in will define the following command aliases:

 * `SessionOpen` is an alias for `OpenSession`
 * `SessionView` is an alias for `ViewSession`
 * `SessionSave` is an alias for `SaveSession`
 * `SessionDelete` is an alias for `DeleteSession`
 * `SessionClose` is an alias for `CloseSession`

Then there are the command aliases for tab scoped sessions:

 * `SessionTabOpen` is an alias for `OpenTabSession`
 * `SessionTabSave` is an alias for `SaveTabSession`
 * `SessionTabAppend` is an alias for `AppendTabSession`
 * `SessionTabClose` is an alias for `CloseTabSession`

The aliases support tab completion just like the real commands; they're exactly the same except for the names.

When you enable the aliases, the default command names will still be available. If you really don't like them, feel free to delete them using [:delcommand] [delcommand].

### The `g:session_menu` option

By default the plug-in installs a top level menu. If you don't like this you can disable it by adding the following line to your [vimrc script] [vimrc]:

    :let g:session_menu = 0

### The `g:session_name_suggestion_function` option

The default completion of the `:SaveSession` command is based on the names of the existing sessions. You can add your own suggestions using this option by setting the option to the name of a Vim script function. By default this option is set to an example function that suggests the name of the current git or Mercurial feature branch (when you're working in a version control repository).

### The `g:loaded_session` option

This variable isn't really an option but if you want to avoid loading the vim-session plug-in you can set this variable to any value in your [vimrc script] [vimrc]:

    :let g:loaded_session = 1

## Compatibility with other plug-ins

Vim's [:mksession][mksession] command isn't really compatible with plug-ins that create buffers with generated content and because of this the vim-session plug-in includes specific workarounds for a couple of popular plug-ins:

 * [BufExplorer](http://www.vim.org/scripts/script.php?script_id=42), [Conque Shell](http://www.vim.org/scripts/script.php?script_id=2771), [NERD tree](http://www.vim.org/scripts/script.php?script_id=1658), [Project](http://www.vim.org/scripts/script.php?script_id=69) and [taglist](http://www.vim.org/scripts/script.php?script_id=273) windows are supported;
 * When [shell.vim](http://peterodding.com/code/vim/shell/) is installed Vim's full-screen state is persisted;
 * The [netrw](http://vimdoc.sourceforge.net/htmldoc/pi_netrw.html#netrw-start) plug-in supports sessions out of the box.

If your favorite plug-in doesn't work with the vim-session plug-in drop me a mail and I'll see what I can do. Please include a link to the plug-in in your e-mail so that I can install and test the plug-in.

## Known issues

Recently this plug-in switched from reimplementing [:mksession][mksession] to actually using it because this was the only way to support complex split window layouts. Only after making this change did I realize [:mksession][mksession] doesn't support [quickfix](http://vimdoc.sourceforge.net/htmldoc/quickfix.html#quickfix) and [location list](http://vimdoc.sourceforge.net/htmldoc/quickfix.html#location-list) windows and of course it turns out that bolting on support for these after the fact is going to complicate the plug-in significantly (in other words, I'm working on it but it might take a while...)

## Function reference

<!-- Start of generated documentation -->

The documentation of the 37 functions below was extracted from
2 Vim scripts on July  7, 2014 at 01:17.

### Public API for the vim-session plug-in

#### The `xolox#session#save_session()` function

Save the current Vim editing session to a Vim script using the
[:mksession] [] command and some additional Vim magic provided by the
vim-session plug-in. When the generated session script is later sourced
using the [:source] [] command (possibly in another process or even on
another machine) it will restore the editing session to its previous state
(provided all of the files involved in the session are still there at
their original locations).

The first argument is expected to be a list, it will be extended with the
lines to be added to the session script. The second argument is expected
to be the filename under which the script will later be saved (it's
embedded in a comment at the top of the script).

[:mksession]: http://vimdoc.sourceforge.net/htmldoc/starting.html#:mksession
[:source]: http://vimdoc.sourceforge.net/htmldoc/repeat.html#:source

#### The `xolox#session#save_globals()` function

Serialize the values of the global variables configured by the user with
the `g:session_persist_globals` option. The first argument is expected to
be a list, it will be extended with the lines to be added to the session
script.

#### The `xolox#session#save_features()` function

Save the current state of the following Vim features:

- Whether syntax highlighting is enabled (`:syntax on`)
- Whether file type detection is enabled (`:filetype on`)
- Whether file type plug-ins are enabled (`:filetype plugin on`)
- Whether file type indent plug-ins are enabled (`:filetype indent on`)

The first argument is expected to be a list, it will be extended with the
lines to be added to the session script.

#### The `xolox#session#save_colors()` function

Save the current color scheme and background color. The first argument is
expected to be a list, it will be extended with the lines to be added to
the session script.

#### The `xolox#session#save_fullscreen()` function

Save the full screen state of Vim. This function provides integration
between my [vim-session] [] and [vim-shell] [] plug-ins. The first
argument is expected to be a list, it will be extended with the lines to
be added to the session script.

[vim-session]: http://peterodding.com/code/vim/session
[vim-shell]: http://peterodding.com/code/vim/shell

#### The `xolox#session#save_qflist()` function

Save the contents of the quick-fix list. The first argument is expected to
be a list, it will be extended with the lines to be added to the session
script.

#### The `xolox#session#save_state()` function

Wrapper for the [:mksession] [] command that slightly massages the
generated Vim script to get rid of some strange quirks in the way Vim
generates sessions. Also implements support for buffers with content that
was generated by other Vim plug-ins. The first argument is expected to
be a list, it will be extended with the lines to be added to the session
script.

#### The `xolox#session#save_special_windows()` function

Implements support for buffers with content that was generated by other
Vim plug-ins. The first argument is expected to be a list, it will be
extended with the lines to be added to the session script.

#### The `xolox#session#auto_load()` function

Automatically load the default or last used session when Vim starts.
Normally called by the [VimEnter] [] automatic command event.

[VimEnter]: http://vimdoc.sourceforge.net/htmldoc/autocmd.html#VimEnter

#### The `xolox#session#auto_save()` function

Automatically save the current editing session when Vim is closed.
Normally called by the [VimLeavePre] [] automatic command event.

[VimLeavePre]: http://vimdoc.sourceforge.net/htmldoc/autocmd.html#VimLeavePre

#### The `xolox#session#auto_save_periodic()` function

Automatically saves the current editing session every few minutes.
Normally called by the [CursorHold] [] and [CursorHoldI] [] automatic
command events.

[CursorHold]: http://vimdoc.sourceforge.net/htmldoc/autocmd.html#CursorHold
[CursorHoldI]: http://vimdoc.sourceforge.net/htmldoc/autocmd.html#CursorHoldI

#### The `xolox#session#auto_unlock()` function

Automatically unlock all sessions when Vim quits. Normally called by the
[VimLeavePre] [] automatic command event.

[VimLeavePre]: http://vimdoc.sourceforge.net/htmldoc/autocmd.html#VimLeavePre

#### The `xolox#session#prompt_for_name()` function

Prompt the user to select one of the existing sessions. The first argument
is expected to be a string describing what will be done to the session
once it's been selected. Returns the name of the selected session as a
string. If no session is selected an empty string is returned. Here's
an example of what the prompt looks like:

    :call xolox#session#prompt_for_name('trash')

    Please select the session to trash:

     1. first-session
     2. second-session
     3. third-session

    Type number and <Enter> or click with mouse (empty cancels):

If only a single session exists there's nothing to choose from so the name
of that session will be returned directly, without prompting the user.

#### The `xolox#session#name_to_path()` function

Convert the name of a session (the first argument, expected to be a
string) to an absolute pathname. Any special characters in the session
name will be encoded using URL encoding. This means you're free to use
whatever naming conventions you like (regardless of special characters
like slashes). Returns a string.

#### The `xolox#session#path_to_name()` function

Convert the absolute pathname of a session script (the first argument,
expected to be a string) to a session name. This function assumes the
absolute pathname refers to the configured session directory, but it does
not check for it nor does it require it (it simple takes the base name
of the absolute pathname of the session script and decodes it). Returns a
string.

#### The `xolox#session#get_names()` function

Get the names of all available sessions. This scans the directory
configured with `g:session_directory` for files that end with the suffix
configured with `g:session_extension`, takes the base name of each file
and decodes any URL encoded characters. Returns a list of strings.

If the first argument is true (1) then the user defined function
configured with `g:session_name_suggestion_function` is called to find
suggested session names, which are prefixed to the list of available
sessions, otherwise the argument should be false (0).

#### The `xolox#session#complete_names()` function

Completion function for user defined Vim commands. Used by commands like
`:OpenSession` and `:DeleteSession`  (but not `:SaveSession`) to support
user friendly completion.

#### The `xolox#session#complete_names_with_suggestions()` function

Completion function for the Vim command `:SaveSession`.

#### The `xolox#session#is_tab_scoped()` function

Determine whether the current session is tab scoped or global. Returns 1
(true) when the session is tab scoped, 0 (false) otherwise.

#### The `xolox#session#find_current_session()` function

Find the name of the current tab scoped or global session. Returns a
string. If no session is active an empty string is returned.

#### The `xolox#session#get_label()` function

Get a human readable label based on the scope (tab scoped or global) and
name of a session. The first argument is the name (a string) and the
second argument is a boolean indicating the scope of the session; 1 (true)
means tab scoped and 0 (false) means global scope. Returns a string.

#### The `xolox#session#options_include()` function

Check whether Vim's [sessionoptions] [] option includes the keyword given
as the first argument (expected to be a string). Returns 1 (true) when it
does, 0 (false) otherwise.

[sessionoptions]: http://vimdoc.sourceforge.net/htmldoc/options.html#'sessionoptions'

#### The `xolox#session#include_tabs()` function

Check whether Vim's [sessionoptions] [] option includes the `tabpages`
keyword. Returns 1 (true) when it does, 0 (false) otherwise.

#### The `xolox#session#change_tab_options()` function

Temporarily change Vim's [sessionoptions] [] option so we can save a tab
scoped session. Saves a copy of the original value to be restored later.

#### The `xolox#session#restore_tab_options()` function

Restore the original value of Vim's [sessionoptions] [] option.

### Example function for session name suggestions

#### The `xolox#session#suggestions#vcs_feature_branch()` function

This function implements an example of a function that can be used with
the `g:session_name_suggestion_function` option. It finds the name of the
current git or Mercurial feature branch (if any) and suggests this name as
the name for the session that is being saved with :SaveSession. Returns a
list with one string on success and an empty list on failure.

<!-- End of generated documentation -->

## Contact

If you have questions, bug reports, suggestions, etc. the author can be contacted at <peter@peterodding.com>. The latest version is available at <http://peterodding.com/code/vim/session/> and <http://github.com/xolox/vim-session>. If you like the script please vote for it on [Vim Online](http://www.vim.org/scripts/script.php?script_id=3150).

## License

This software is licensed under the [MIT license](http://en.wikipedia.org/wiki/MIT_License).  
© 2014 Peter Odding &lt;<peter@peterodding.com>&gt; and Ingo Karkat.

Thanks go out to everyone who has helped to improve the vim-session plug-in (whether through pull requests, bug reports or personal e-mails).

## Sample session script

Here's an example session script generated by the vim-session plug-in while I was editing the plug-in itself in Vim:

    " ~/.vim/sessions/example.vim: Vim session script.
    " Created by session.vim on 30 August 2010 at 05:26:28.
    " Open this file in Vim and run :source % to restore your session.

    set guioptions=aegit
    set guifont=Monaco\ 13
    if exists('g:syntax_on') != 1 | syntax on | endif
    if exists('g:did_load_filetypes') != 1 | filetype on | endif
    if exists('g:did_load_ftplugin') != 1 | filetype plugin on | endif
    if exists('g:did_indent_on') != 1 | filetype indent on | endif
    if !exists('g:colors_name') || g:colors_name != 'slate' | colorscheme slate | endif
    call setqflist([])
    let SessionLoad = 1
    if &cp | set nocp | endif
    let s:so_save = &so | let s:siso_save = &siso | set so=0 siso=0
    let v:this_session=expand("<sfile>:p")
    silent only
    cd ~/Development/Vim/vim-session
    if expand('%') == '' && !&modified && line('$') <= 1 && getline(1) == ''
      let s:wipebuf = bufnr('%')
    endif
    set shortmess=aoO
    badd +473 ~/Development/Vim/vim-session/autoload.vim
    badd +1 ~/Development/Vim/vim-session/README.md
    badd +1 ~/Development/Vim/vim-session/session.vim
    badd +1 ~/Development/Vim/vim-session/TODO.md
    set lines=43 columns=167
    edit ~/Development/Vim/vim-session/README.md
    set splitbelow splitright
    set nosplitbelow
    set nosplitright
    wincmd t
    set winheight=1 winwidth=1
    argglobal
    let s:l = 28 - ((27 * winheight(0) + 21) / 42)
    if s:l < 1 | let s:l = 1 | endif
    exe s:l
    normal! zt
    28
    normal! 0
    tabedit ~/Development/Vim/vim-session/TODO.md
    set splitbelow splitright
    set nosplitbelow
    set nosplitright
    wincmd t
    set winheight=1 winwidth=1
    argglobal
    let s:l = 6 - ((5 * winheight(0) + 21) / 42)
    if s:l < 1 | let s:l = 1 | endif
    exe s:l
    normal! zt
    6
    normal! 0
    tabedit ~/Development/Vim/vim-session/session.vim
    set splitbelow splitright
    set nosplitbelow
    set nosplitright
    wincmd t
    set winheight=1 winwidth=1
    argglobal
    let s:l = 17 - ((16 * winheight(0) + 21) / 42)
    if s:l < 1 | let s:l = 1 | endif
    exe s:l
    normal! zt
    17
    normal! 014l
    tabedit ~/Development/Vim/vim-session/autoload.vim
    set splitbelow splitright
    set nosplitbelow
    set nosplitright
    wincmd t
    set winheight=1 winwidth=1
    argglobal
    let s:l = 473 - ((41 * winheight(0) + 21) / 42)
    if s:l < 1 | let s:l = 1 | endif
    exe s:l
    normal! zt
    473
    normal! 018l
    tabnext 4
    if exists('s:wipebuf')
      silent exe 'bwipe ' . s:wipebuf
    endif
    unlet! s:wipebuf
    set winheight=1 winwidth=1 shortmess=filnxtToO
    let s:sx = expand("<sfile>:p:r")."x.vim"
    if file_readable(s:sx)
      exe "source " . fnameescape(s:sx)
    endif
    let &so = s:so_save | let &siso = s:siso_save
    doautoall SessionLoadPost
    unlet SessionLoad


[bg]: http://vimdoc.sourceforge.net/htmldoc/options.html#'background'
[delcommand]: http://vimdoc.sourceforge.net/htmldoc/map.html#:delcommand
[download-misc]: http://peterodding.com/code/vim/downloads/misc.zip
[download-session]: http://peterodding.com/code/vim/downloads/session.zip
[github-misc]: http://github.com/xolox/vim-misc
[github-session]: http://github.com/xolox/vim-session
[mksession]: http://vimdoc.sourceforge.net/htmldoc/starting.html#:mksession
[pathogen]: http://www.vim.org/scripts/script.php?script_id=2332
[sessionoptions]: http://vimdoc.sourceforge.net/htmldoc/options.html#%27sessionoptions%27
[source]: http://vimdoc.sourceforge.net/htmldoc/repeat.html#:source
[tabnew]: http://vimdoc.sourceforge.net/htmldoc/tabpage.html#:tabnew
[vimrc]: http://vimdoc.sourceforge.net/htmldoc/starting.html#vimrc
[vundle]: https://github.com/gmarik/vundle
