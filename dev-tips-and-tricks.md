# Tips and Tricks

A collection of little things that I have found to be useful and I don't
want to forget.

## Terminal (iTerm)

### Suspend a process
Useful when you want to check another document or run another command, but don't
want to have to exit out of vim or close all you panes.

Suspend the process,
<kbd>control</kbd> + <kbd>z</kbd>

Bring the process back to the forground,

```
% fg
```

Suspended multiple things and don't know what to do,

```
% ps
```

will list current background processes.
Then you can use,

```
% fg 2
```

to restore the second background process.
