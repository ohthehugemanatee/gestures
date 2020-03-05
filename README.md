
# Table of Contents

1.  [Motivation](#orgd19f4c1)
2.  [Features](#orge3dc242)
3.  [Dependencies](#org2ad86f8)
4.  [Installation/Uninstallation](#orgd57d887)
5.  [Customization](#org40c2a4c)
6.  [The Code](#org044358a)
7.  [Alternatives](#orgd9dd9d6)
8.  [TODOS](#org0482173)
    1.  [add features <code>[0/2]</code>](#org467ae51)
    2.  [enrich readme](#org2f2e458)
    3.  [Write script to fulfill dependencies automatically](#org4f36646)
    4.  [Implement C++ version](#orge82e6fc)

![img](gestures.gif "demonstrating fluid gestures, five finger gestures, tap gestures and touchscreen gestures")


<a id="orgd19f4c1"></a>

# Motivation

-   I wanted an application that will allow me to fluidly express my intent through the touchpad and touchscreen.
-   I felt shackled by one-shot gestures from [libinput-gestures](https://github.com/bulletmark/libinput-gestures) but from it I found the utility of eight-directional gestures.
-   I felt [fusuma](https://github.com/iberianpig/fusuma) was lacking Although it was more fluid.
-   I really missed keeping my fingers moving across the touchpad to switch applications.
-   I wanted tap gestures.
-   I wanted five finger gestures.
-   I wanted usable gestures for my touchscreen.

So I wrote this.


<a id="orge3dc242"></a>

# Features

-   **vanilla gestures:** features present in [libinput-gestures](https://github.com/bulletmark/libinput-gestures) and [fusuma](https://github.com/iberianpig/fusuma).
-   **5 finger gestures:** 

This means that you can place 5 fingers on the touchpad and that is recognized as another class of gestures. BTN<sub>TOOL</sub><sub>QUINTTAP</sub> must supported. This feature must supported by your driver + touchpad.
check by running `evtest /dev/input/$(cat /proc/bus/input/devices | grep -iA 5 'touchpad' |grep -oP 'event[0-9]+') | grep BTN_TOOL_QUINTTAP`. if it prints a line containing `"BTN_TOOL_QUINTTAP"`, your touchpad+driver support it.

-   **tap gestures:** 

This means that tapping the touchpad with 4 or 5 fingers is recognized as a gesture.

-   **Touchscreen gestures:** 

Extend gestures to touchscreen.

-   **Fluid gestures:** 

Allow the user to do different things without raising their fingers from the touchpad. This takes advantage of the fact that gesture execution is separated into 3 parts, start, update, and end, by doing complementary actions in each.

For example, assume the shortcut to switch windows is "CTRL + ALT" + direction, where direction is "LEFT", "RIGHT", "UP", "DOWN".

to switch windows fluidly, "CTRL + ALT" are held down programmatically on a start gesture event, then depending on which direction the user is swiping while fingers are still on the touchpad, dynamically generate direction commands. Then when the user raises their fingers, which is an end gesture event, the "CTRL + ALT" are released programmatically.


<a id="org2ad86f8"></a>

# Dependencies

-   **python dependencies:** subprocess, pathlib, shlex, threading, queue, time, os, sys, math
-   **\*nix dependencies:** cat, grep
-   **dependencies:** stdbuf, evtest
    -   **evtest:** will maybe replaced by evemu-record in the future.
    -   **daemonize:** using `& disown` should work as well but this is a sure way to detach and run this on a global scale.
-   **default dependencies (if running default configuration):** -   **evemu:** need evemu-do (alternative to xdotool that I wrote) in $PATH.


<a id="orgd57d887"></a>

# Installation/Uninstallation

-   **installation:** -   run ./install.sh
        -   it should handle most things.
        -   may need to install `daemonize` by hand. If on Arch, I recommend `daemonize-git` from AUR.
        -   may want to look at where it places things and if that meets your setup.
        -   what you truly need from this repo is gestures.config, gestures, getConfig.py. Everything else is just dependencies.
-   **uninstallation:** -   run ./uninstall.sh
    -   removes everything except that what was installed by the package manager. To uninstall those, remove `evtest` and `daemonize`.


<a id="org40c2a4c"></a>

# Customization

-   **default customization:** -   the default customization is my config.
    -   uses extensively `evemu_do`, a script I wrote to replace `xdotool`. Much less buggy and also works on wayland.
    -   `evemu_do` works much like xdotool but only for keyboard inputs.
        -   `evemu_do tab` presses tab (also supports `evemu_do key tab`)
        -   `evemu_do keydown tab` holds down tab
        -   `evemu_do keyup tab` de-presses tab
    -   currently works by dumping events in the first keyboard it finds under /proc/bus/input/devices.
        -   may look into creating a keyboard device for it to dump all its events on.
    -   underneath it uses `evemu-event`, which is part of the `evemu` toolkit.

-   **currently customizable:** -   swipe, pinch
    -   3,4,5 finger start and end gestures
    -   specific gestures for touchpad and touchscreen
-   **example:** {'touchpad': 
         {'swipe': {
             '3': {
                 'l' : {'start': ['evemu_do alt down', 'evemu_do tab'], 'update': {'l': [], 'r': [], 'u': [], 'd': [], 'lu': [], 'rd': [], 'ld': [], 'ru': []}, 'end': ['evemu_do alt up'], 'rep': ''}
             }
         }
         }
        }
-   **breakdown:** -   **(touchscreen, touchpad):** -   make a set of gestures apply to touchpad or touchscreen
    -   **(swipe,pinch):** -   define if the gesture is a swipe or a pinch
    -   **(3,4,5):** -   define the number of fingers to activate the gesture
    -   **('t', 'l', 'r',&#x2026;,'ru'):** define tap and the 8 directions a swipe can be in.
    -   **('i', 'o'):** define pinch in and pinch out.
    -   **(start,end):** -   what to do when the gesture starts or ends.
    -   **(slated for a future update):** -   **(update):** -   what to do when the gesture is on going. going to start out with just 4 directions as that suffices my needs (and probably most others) but will expand to 8 directional configuration should there be demand.
        -   **(rep):** -   how frequently is gesture update run. can make this directional as well, but don't have plans for that yet.
        -   **(device level tag):** -   can already have gestures apply to touchscreen or touchpad. the extension to specify what device a specific set of gestures apply to.


<a id="org044358a"></a>

# The Code

-   may need to adjust the screen size and touchpad calibration. This can be automated by looking at the dimensions as evtest is called.
-   the knobs are as follows

TOUCHPAD<sub>CALIBRATION</sub> = 1 # scaling down for touchpad movements
TOUCHSCREEN<sub>CALIBRATION</sub> = 2 # scaling down for touchscreen movements

DECISION = 450 # sufficient movement to make decision on direction
PINCH<sub>DECISION</sub> = 160 #seems like x<sub>cum</sub> and y<sub>cum</sub> should got to around 0 if finges moved symetrically in or out  #sufficient momvent to make pinch

ANGLE = 70 #x/y angle cleance
CLEARANCE = 10#clearance for not intrepreting swipes between diagonal and horizontal or vertical

DEBOUNCE = 0.04  #sleep for now 40 ms, fastest tap around 25 ms , gotten from new<sub>touch</sub>, touchpad data. in practice works well.
THRESHOLD = 150 # threashold to be considered a move, squared sum of x and y
PINCH<sub>THRESHOLD</sub> = 100

REP<sub>THRES</sub> = 0.2 #need to break this TIME before REP engage
REP = 350 # for 3 finger stuff
REP<sub>3</sub> = 150 # for 3 finger stuff
REP<sub>4x</sub>= 450 # for 4 finger x, was having issue with horizontal swipes overstepping but vertical ones being perdicatable
REP<sub>4</sub> = 450 # for 4 finger stuff; repeat after this much x,y movement
PINCH<sub>REP</sub> = 40


<a id="orgd9dd9d6"></a>

# Alternatives

-   **[libinput-gestures](https://github.com/bulletmark/libinput-gestures):** -   what I used to use.
    -   Works well, just that the gestures are one-shot, meaning that the command attached to a gesture is executed only once per full swipe.
    -   depends on libinput.
-   **[fusuma](https://github.com/iberianpig/fusuma):** -   Although it doesn't have one-shot limitation, it didn't support commands to run when the gesture begins and ends. This is useful for use-cases like switching applications which require alt-down to be pressed.
    -   didn't support eight-directional gestures.


<a id="org0482173"></a>

# TODOS



<a id="org467ae51"></a>

## TODO add features <code>[0/2]</code>

-   [-]<code>[3/7]</code> enable customization by refactoring code.
    -   [X] commands for gesture start
    -   [X] commands for gesture end
    -   [X] commands for touchscreen
    -   [ ] commands for gesture update
    -   [ ] rep rate
    -   [ ] detach implementation from personal workflow
    -   [ ] more nuanced application of gestures to different attached devices
-   [ ] use [libinput-gestures ](https://github.com/bulletmark/libinput-gestures)config file syntax.
-   [ ] use [fusuma](https://github.com/iberianpig/fusuma) config file syntax.


<a id="org2f2e458"></a>

## DONE enrich readme


<a id="org4f36646"></a>

## DONE Write script to fulfill dependencies automatically


<a id="orge82e6fc"></a>

## TODO Implement C++ version
