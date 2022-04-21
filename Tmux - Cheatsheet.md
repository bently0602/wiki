# tmux cheatsheet

Forked from: [tmux cheatsheet](https://gist.githubusercontent.com/henrik/1967800/raw/f580aa23cbc5cbf1bd96dee1c903784c0e32eca2/tmux_cheatsheet.markdown)

Prefix key: `ctrl+b`

## Sessions from commands

start new:

    tmux

start new with session name:

    tmux new -s myname

list sessions:

    tmux ls
	
attach:

    tmux a  #  (or at, or attach)

attach to named:

    tmux a -t myname

kill session:

    tmux kill-session -t myname

## Sessions inside of tmux

    :new<CR>  new session
    s  list sessions
    $  name session

## Windows (tabs)

    c           new window
    n		next window (carousel like)
    p		previous window (carousel like)
    ,           name window
    w           list windows
    f           find window
    &           kill window
    .           move window - prompted for a new number
    :movew<CR>  move window to the next unused number

## Panes (splits)

    %  horizontal split
    "  vertical split
    
    o  swap panes
    q  show pane numbers
    x  kill pane
    ⍽  space - toggle between layouts

## Window/pane surgery

    :joinp -s :2<CR>  move window 2 into a new pane in the current window
    :joinp -t :1<CR>  move the current pane into a new pane in window 1

* [Move window to pane](http://unix.stackexchange.com/questions/14300/tmux-move-window-to-pane)
* [How to reorder windows](http://superuser.com/questions/343572/tmux-how-do-i-reorder-my-windows)

## Misc

    d  detach
    t  big clock
    ?  list shortcuts
    :  prompt

## Troubleshoot

- Resize screen from previous session's screen size

	:resize-window -A

Resources:

* [cheat sheet](http://cheat.errtheblog.com/s/tmux/)
* [StackOverflow - Resize](https://stackoverflow.com/questions/7814612/is-there-any-way-to-redraw-tmux-window-when-switching-smaller-monitor-to-bigger)
 
Notes:

* You can cmd+click URLs to open in iTerm.

TODO:

* Conf copy mode to use system clipboard. See PragProg book.
