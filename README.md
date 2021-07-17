keyboard
Take full control of your keyboard with this small Python library. Hook global events, register hotkeys, simulate key presses and much more.

Features
Global event hook on all keyboards (captures keys regardless of focus).
Listen and send keyboard events.
Works with Windows and Linux (requires sudo), with experimental OS X support (thanks @glitchassassin!).
Pure Python, no C modules to be compiled.
Zero dependencies. Trivial to install and deploy, just copy the files.
Python 2 and 3.
Complex hotkey support (e.g. ctrl+shift+m, ctrl+space) with controllable timeout.
Includes high level API (e.g. record and play, add_abbreviation).
Maps keys as they actually are in your layout, with full internationalization support (e.g. Ctrl+ç).
Events automatically captured in separate thread, doesn't block main program.
Tested and documented.
Doesn't break accented dead keys (I'm looking at you, pyHook).
Mouse support available via project mouse (pip install mouse).
Usage
Install the PyPI package:

pip install keyboard
or clone the repository (no installation required, source files are sufficient):

git clone https://github.com/boppreh/keyboard
or download and extract the zip into your project folder.

Then check the API docs below to see what features are available.

Example
Use as library:

import keyboard

keyboard.press_and_release('shift+s, space')

keyboard.write('The quick brown fox jumps over the lazy dog.')

keyboard.add_hotkey('ctrl+shift+a', print, args=('triggered', 'hotkey'))

# Press PAGE UP then PAGE DOWN to type "foobar".
keyboard.add_hotkey('page up, page down', lambda: keyboard.write('foobar'))

# Blocks until you press esc.
keyboard.wait('esc')

# Record events until 'esc' is pressed.
recorded = keyboard.record(until='esc')
# Then replay back at three times the speed.
keyboard.play(recorded, speed_factor=3)

# Type @@ then press space to replace with abbreviation.
keyboard.add_abbreviation('@@', 'my.long.email@example.com')

# Block forever, like `while True`.
keyboard.wait()
Use as standalone module:

# Save JSON events to a file until interrupted:
python -m keyboard > events.txt

cat events.txt
# {"event_type": "down", "scan_code": 25, "name": "p", "time": 1622447562.2994788, "is_keypad": false}
# {"event_type": "up", "scan_code": 25, "name": "p", "time": 1622447562.431007, "is_keypad": false}
# ...

# Replay events
python -m keyboard < events.txt
Known limitations:
Events generated under Windows don't report device id (event.device == None). #21
Media keys on Linux may appear nameless (scan-code only) or not at all. #20
Key suppression/blocking only available on Windows. #22
To avoid depending on X, the Linux parts reads raw device files (/dev/input/input*) but this requires root.
Other applications, such as some games, may register hooks that swallow all key events. In this case keyboard will be unable to report events.
This program makes no attempt to hide itself, so don't use it for keyloggers or online gaming bots. Be responsible.
SSH connections forward only the text typed, not keyboard events. Therefore if you connect to a server or Raspberry PI that is running keyboard via SSH, the server will not detect your key events.
Common patterns and mistakes
Preventing the program from closing
import keyboard
keyboard.add_hotkey('space', lambda: print('space was pressed!'))
# If the program finishes, the hotkey is not in effect anymore.

# Don't do this! This will use 100% of your CPU.
#while True: pass

# Use this instead
keyboard.wait()

# or this
import time
while True:
    time.sleep(1000000)
Waiting for a key press one time
import keyboard

# Don't do this! This will use 100% of your CPU until you press the key.
#
#while not keyboard.is_pressed('space'):
#    continue
#print('space was pressed, continuing...')

# Do this instead
keyboard.wait('space')
print('space was pressed, continuing...')
Repeatedly waiting for a key press
import keyboard

# Don't do this!
#
#while True:
#    if keyboard.is_pressed('space'):
#        print('space was pressed!')
#
# This will use 100% of your CPU and print the message many times.

# Do this instead
while True:
    keyboard.wait('space')
    print('space was pressed! Waiting on it again...')

# or this
keyboard.add_hotkey('space', lambda: print('space was pressed!'))
keyboard.wait()
Invoking code when an event happens
import keyboard

# Don't do this! This will call `print('space')` immediately then fail when the key is actually pressed.
#keyboard.add_hotkey('space', print('space was pressed'))

# Do this instead
keyboard.add_hotkey('space', lambda: print('space was pressed'))

# or this
def on_space():
    print('space was pressed')
keyboard.add_hotkey('space', on_space)
'Press any key to continue'
# Don't do this! The `keyboard` module is meant for global events, even when your program is not in focus.
#import keyboard
#print('Press any key to continue...')
#keyboard.get_event()

# Do this instead
input('Press enter to continue...')

# Or one of the suggestions from here
# https://stackoverflow.com/questions/983354/how-to-make-a-script-wait-for-a-pressed-key
API
Table of Contents
keyboard.KEY_DOWN
keyboard.KEY_UP
keyboard.KeyboardEvent
keyboard.all_modifiers
keyboard.sided_modifiers
keyboard.version
keyboard.is_modifier
keyboard.key_to_scan_codes
keyboard.parse_hotkey
keyboard.send (aliases: press_and_release)
keyboard.press
keyboard.release
keyboard.is_pressed
keyboard.call_later
keyboard.hook
keyboard.on_press
keyboard.on_release
keyboard.hook_key
keyboard.on_press_key
keyboard.on_release_key
keyboard.unhook (aliases: unblock_key, unhook_key, unremap_key)
keyboard.unhook_all
keyboard.block_key
keyboard.remap_key
keyboard.parse_hotkey_combinations
keyboard.add_hotkey (aliases: register_hotkey)
keyboard.remove_hotkey (aliases: clear_hotkey, unregister_hotkey, unremap_hotkey)
keyboard.unhook_all_hotkeys (aliases: clear_all_hotkeys, remove_all_hotkeys, unregister_all_hotkeys)
keyboard.remap_hotkey
keyboard.stash_state
keyboard.restore_state
keyboard.restore_modifiers
keyboard.write
keyboard.wait
keyboard.get_hotkey_name
keyboard.read_event
keyboard.read_key
keyboard.read_hotkey
keyboard.get_typed_strings
keyboard.start_recording
keyboard.stop_recording
keyboard.record
keyboard.play (aliases: replay)
keyboard.add_word_listener (aliases: register_word_listener)
keyboard.remove_word_listener (aliases: remove_abbreviation)
keyboard.add_abbreviation (aliases: register_abbreviation)
keyboard.normalize_name
keyboard.KEY_DOWN
= 'down'
keyboard.KEY_UP
= 'up'
class keyboard.KeyboardEvent
KeyboardEvent.device
KeyboardEvent.event_type
KeyboardEvent.is_keypad
KeyboardEvent.modifiers
KeyboardEvent.name
KeyboardEvent.scan_code
KeyboardEvent.time
KeyboardEvent.to_json(self, ensure_ascii=False)
[source]

keyboard.all_modifiers
= {'alt', 'alt gr', 'ctrl', 'left alt', 'left ctrl', 'left shift', 'left windows', 'right alt', 'right ctrl', 'right shift', 'right windows', 'shift', 'windows'}
keyboard.sided_modifiers
= {'alt', 'ctrl', 'shift', 'windows'}
keyboard.version
= '0.13.5'
keyboard.is_modifier(key)
[source]

Returns True if key is a scan code or name of a modifier key.

keyboard.key_to_scan_codes(key, error_if_missing=True)
[source]

Returns a list of scan codes associated with this key (name or scan code).

keyboard.parse_hotkey(hotkey)
[source]

Parses a user-provided hotkey into nested tuples representing the parsed structure, with the bottom values being lists of scan codes. Also accepts raw scan codes, which are then wrapped in the required number of nestings.

Example:

parse_hotkey("alt+shift+a, alt+b, c")
#    Keys:    ^~^ ^~~~^ ^  ^~^ ^  ^
#    Steps:   ^~~~~~~~~~^  ^~~~^  ^

# ((alt_codes, shift_codes, a_codes), (alt_codes, b_codes), (c_codes,))
keyboard.send(hotkey, do_press=True, do_release=True)
[source]

Sends OS events that perform the given hotkey hotkey.

hotkey can be either a scan code (e.g. 57 for space), single key (e.g. 'space') or multi-key, multi-step hotkey (e.g. 'alt+F4, enter').
do_press if true then press events are sent. Defaults to True.
do_release if true then release events are sent. Defaults to True.
send(57)
send('ctrl+alt+del')
send('alt+F4, enter')
send('shift+s')
Note: keys are released in the opposite order they were pressed.

keyboard.press(hotkey)
[source]

Presses and holds down a hotkey (see send).

keyboard.release(hotkey)
[source]

Releases a hotkey (see send).

keyboard.is_pressed(hotkey)
[source]

Returns True if the key is pressed.

is_pressed(57) #-> True
is_pressed('space') #-> True
is_pressed('ctrl+space') #-> True
keyboard.call_later(fn, args=(), delay=0.001)
[source]

Calls the provided function in a new thread after waiting some time. Useful for giving the system some time to process an event, without blocking the current execution flow.

keyboard.hook(callback, suppress=False, on_remove=<lambda>)
[source]

Installs a global listener on all available keyboards, invoking callback each time a key is pressed or released.

The event passed to the callback is of type keyboard.KeyboardEvent, with the following attributes:

name: an Unicode representation of the character (e.g. "&") or description (e.g. "space"). The name is always lower-case.
scan_code: number representing the physical key, e.g. 55.
time: timestamp of the time the event occurred, with as much precision as given by the OS.
Returns the given callback for easier development.

keyboard.on_press(callback, suppress=False)
[source]

Invokes callback for every KEY_DOWN event. For details see hook.

keyboard.on_release(callback, suppress=False)
[source]

Invokes callback for every KEY_UP event. For details see hook.

keyboard.hook_key(key, callback, suppress=False)
[source]

Hooks key up and key down events for a single key. Returns the event handler created. To remove a hooked key use unhook_key(key) or unhook_key(handler).

Note: this function shares state with hotkeys, so clear_all_hotkeys affects it as well.

keyboard.on_press_key(key, callback, suppress=False)
[source]

Invokes callback for KEY_DOWN event related to the given key. For details see hook.

keyboard.on_release_key(key, callback, suppress=False)
[source]

Invokes callback for KEY_UP event related to the given key. For details see hook.

keyboard.unhook(remove)
[source]

Removes a previously added hook, either by callback or by the return value of hook.

keyboard.unhook_all()
[source]

Removes all keyboard hooks in use, including hotkeys, abbreviations, word listeners, recorders and waits.

keyboard.block_key(key)
[source]

Suppresses all key events of the given key, regardless of modifiers.

keyboard.remap_key(src, dst)
[source]

Whenever the key src is pressed or released, regardless of modifiers, press or release the hotkey dst instead.

keyboard.parse_hotkey_combinations(hotkey)
[source]

Parses a user-provided hotkey. Differently from parse_hotkey, instead of each step being a list of the different scan codes for each key, each step is a list of all possible combinations of those scan codes.

keyboard.add_hotkey(hotkey, callback, args=(), suppress=False, timeout=1, trigger_on_release=False)
[source]

Invokes a callback every time a hotkey is pressed. The hotkey must be in the format ctrl+shift+a, s. This would trigger when the user holds ctrl, shift and "a" at once, releases, and then presses "s". To represent literal commas, pluses, and spaces, use their names ('comma', 'plus', 'space').

args is an optional list of arguments to passed to the callback during each invocation.
suppress defines if successful triggers should block the keys from being sent to other programs.
timeout is the amount of seconds allowed to pass between key presses.
trigger_on_release if true, the callback is invoked on key release instead of key press.
The event handler function is returned. To remove a hotkey call remove_hotkey(hotkey) or remove_hotkey(handler). before the hotkey state is reset.

Note: hotkeys are activated when the last key is pressed, not released. Note: the callback is executed in a separate thread, asynchronously. For an example of how to use a callback synchronously, see wait.

Examples:

# Different but equivalent ways to listen for a spacebar key press.
add_hotkey(' ', print, args=['space was pressed'])
add_hotkey('space', print, args=['space was pressed'])
add_hotkey('Space', print, args=['space was pressed'])
# Here 57 represents the keyboard code for spacebar; so you will be
# pressing 'spacebar', not '57' to activate the print function.
add_hotkey(57, print, args=['space was pressed'])

add_hotkey('ctrl+q', quit)
add_hotkey('ctrl+alt+enter, space', some_callback)
keyboard.remove_hotkey(hotkey_or_callback)
[source]

Removes a previously hooked hotkey. Must be called with the value returned by add_hotkey.

keyboard.unhook_all_hotkeys()
[source]

Removes all keyboard hotkeys in use, including abbreviations, word listeners, recorders and waits.

keyboard.remap_hotkey(src, dst, suppress=True, trigger_on_release=False)
[source]

Whenever the hotkey src is pressed, suppress it and send dst instead.

Example:

remap('alt+w', 'ctrl+up')
keyboard.stash_state()
[source]

Builds a list of all currently pressed scan codes, releases them and returns the list. Pairs well with restore_state and restore_modifiers.

keyboard.restore_state(scan_codes)
[source]

Given a list of scan_codes ensures these keys, and only these keys, are pressed. Pairs well with stash_state, alternative to restore_modifiers.

keyboard.restore_modifiers(scan_codes)
[source]

Like restore_state, but only restores modifier keys.

keyboard.write(text, delay=0, restore_state_after=True, exact=None)
[source]

Sends artificial keyboard events to the OS, simulating the typing of a given text. Characters not available on the keyboard are typed as explicit unicode characters using OS-specific functionality, such as alt+codepoint.

To ensure text integrity, all currently pressed keys are released before the text is typed, and modifiers are restored afterwards.

delay is the number of seconds to wait between keypresses, defaults to no delay.
restore_state_after can be used to restore the state of pressed keys after the text is typed, i.e. presses the keys that were released at the beginning. Defaults to True.
exact forces typing all characters as explicit unicode (e.g. alt+codepoint or special events). If None, uses platform-specific suggested value.
keyboard.wait(hotkey=None, suppress=False, trigger_on_release=False)
[source]

Blocks the program execution until the given hotkey is pressed or, if given no parameters, blocks forever.

keyboard.get_hotkey_name(names=None)
[source]

Returns a string representation of hotkey from the given key names, or the currently pressed keys if not given. This function:

normalizes names;
removes "left" and "right" prefixes;
replaces the "+" key name with "plus" to avoid ambiguity;
puts modifier keys first, in a standardized order;
sort remaining keys;
finally, joins everything with "+".
Example:

get_hotkey_name(['+', 'left ctrl', 'shift'])
# "ctrl+shift+plus"
keyboard.read_event(suppress=False)
[source]

Blocks until a keyboard event happens, then returns that event.

keyboard.read_key(suppress=False)
[source]

Blocks until a keyboard event happens, then returns that event's name or, if missing, its scan code.

keyboard.read_hotkey(suppress=True)
[source]

Similar to read_key(), but blocks until the user presses and releases a hotkey (or single key), then returns a string representing the hotkey pressed.

Example:

read_hotkey()
# "ctrl+shift+p"
keyboard.get_typed_strings(events, allow_backspace=True)
[source]

Given a sequence of events, tries to deduce what strings were typed. Strings are separated when a non-textual key is pressed (such as tab or enter). Characters are converted to uppercase according to shift and capslock status. If allow_backspace is True, backspaces remove the last character typed.

This function is a generator, so you can pass an infinite stream of events and convert them to strings in real time.

Note this functions is merely an heuristic. Windows for example keeps per- process keyboard state such as keyboard layout, and this information is not available for our hooks.

get_type_strings(record()) #-> ['This is what', 'I recorded', '']
keyboard.start_recording(recorded_events_queue=None)
[source]

Starts recording all keyboard events into a global variable, or the given queue if any. Returns the queue of events and the hooked function.

Use stop_recording() or unhook(hooked_function) to stop.

keyboard.stop_recording()
[source]

Stops the global recording of events and returns a list of the events captured.

keyboard.record(until='escape', suppress=False, trigger_on_release=False)
[source]

Records all keyboard events from all keyboards until the user presses the given hotkey. Then returns the list of events recorded, of type keyboard.KeyboardEvent. Pairs well with play(events).

Note: this is a blocking function. Note: for more details on the keyboard hook and events see hook.

keyboard.play(events, speed_factor=1.0)
[source]

Plays a sequence of recorded events, maintaining the relative time intervals. If speed_factor is <= 0 then the actions are replayed as fast as the OS allows. Pairs well with record().

Note: the current keyboard state is cleared at the beginning and restored at the end of the function.

keyboard.add_word_listener(word, callback, triggers=['space'], match_suffix=False, timeout=2)
[source]

Invokes a callback every time a sequence of characters is typed (e.g. 'pet') and followed by a trigger key (e.g. space). Modifiers (e.g. alt, ctrl, shift) are ignored.

word the typed text to be matched. E.g. 'pet'.
callback is an argument-less function to be invoked each time the word is typed.
triggers is the list of keys that will cause a match to be checked. If the user presses some key that is not a character (len>1) and not in triggers, the characters so far will be discarded. By default the trigger is only space.
match_suffix defines if endings of words should also be checked instead of only whole words. E.g. if true, typing 'carpet'+space will trigger the listener for 'pet'. Defaults to false, only whole words are checked.
timeout is the maximum number of seconds between typed characters before the current word is discarded. Defaults to 2 seconds.
Returns the event handler created. To remove a word listener use remove_word_listener(word) or remove_word_listener(handler).

Note: all actions are performed on key down. Key up events are ignored. Note: word matches are case sensitive.

keyboard.remove_word_listener(word_or_handler)
[source]

Removes a previously registered word listener. Accepts either the word used during registration (exact string) or the event handler returned by the add_word_listener or add_abbreviation functions.

keyboard.add_abbreviation(source_text, replacement_text, match_suffix=False, timeout=2)
[source]

Registers a hotkey that replaces one typed text with another. For example

add_abbreviation('tm', u'™')
Replaces every "tm" followed by a space with a ™ symbol (and no space). The replacement is done by sending backspace events.

match_suffix defines if endings of words should also be checked instead of only whole words. E.g. if true, typing 'carpet'+space will trigger the listener for 'pet'. Defaults to false, only whole words are checked.
timeout is the maximum number of seconds between typed characters before the current word is discarded. Defaults to 2 seconds.
For more details see add_word_listener.

keyboard.normalize_name(name)
[source]

Given a key name (e.g. "LEFT CONTROL"), clean up the string and convert to the canonical representation (e.g. "left ctrl") if one is known.
