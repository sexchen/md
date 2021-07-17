
keyboard
========

Take full control of your keyboard with this small Python library. Hook global events, register hotkeys, simulate key presses and much more.
使用这个小Python库可以完全控制键盘。钩住全局事件，注册热键，模拟按键等等。

## Features

- **Global event hook** on all keyboards (captures keys regardless of focus).
- **Listen** and **send** keyboard events.
- Works with **Windows** and **Linux** (requires sudo), with experimental **OS X** support (thanks @glitchassassin!).
- **Pure Python**, no C modules to be compiled.
- **Zero dependencies**. Trivial to install and deploy, just copy the files.
- **Python 2 and 3**.
- Complex hotkey support (e.g. `ctrl+shift+m, ctrl+space`) with controllable timeout.
- Includes **high level API** (e.g. [record](#keyboard.record) and [play](#keyboard.play), [add_abbreviation](#keyboard.add_abbreviation)).
- Maps keys as they actually are in your layout, with **full internationalization support** (e.g. `Ctrl+ç`).
- Events automatically captured in separate thread, doesn't block main program.
- Tested and documented.
- Doesn't break accented dead keys (I'm looking at you, pyHook).
- Mouse support available via project [mouse](https://github.com/boppreh/mouse) (`pip install mouse`).

# 特征

- **Global event hook** 在所有键盘上（无论焦点如何捕捉按键）。
- **Listen** and **send** 键盘事件 （监听和发送）
- 使用 **Windows** 和 **Linux** (需要 sudo), 实验性的 **OS X** 支持 (感谢 @glitchassassin!).
- **Pure Python**, 没有要编译的C模块。（纯python）
- **Zero dependencies**. 安装和部署很简单，只需复制文件即可。
- **Python 2 and 3**.
-复杂的热键支持 (例如 `ctrl+shift+m, ctrl+space`) 具有可控超时。
- 包括 **high level API** (例如 [record](#keyboard.record) 和 [play](#keyboard.play), [add_abbreviation](#keyboard.add_abbreviation)).
- 将关键点映射为它们在布局中的实际位置, 具有 **完全国际化支持** (例如 `Ctrl+ç`).
- 事件在单独的线程中自动捕获，不会阻塞主程序。
- 测试和记录。
- 不会破坏重音死键 (我说的就是你, pyHook).
- 鼠标支持通过此项目提供 [mouse](https://github.com/boppreh/mouse) (`pip install mouse`).

## 用法

安装 [PyPI package](https://pypi.python.org/pypi/keyboard/):

    pip install keyboard

或者克隆存储库 (无需安装，源文件足够):

    git clone https://github.com/boppreh/keyboard

或者 [下载并解压缩zip](https://github.com/boppreh/keyboard/archive/master.zip) 到项目文件夹中。

然后检查 [API文档如下](https://github.com/boppreh/keyboard#api) 查看哪些功能可用。


## 例子

Use as library:

```py
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
```

Use as standalone module:

```bash
# Save JSON events to a file until interrupted:
python -m keyboard > events.txt

cat events.txt
# {"event_type": "down", "scan_code": 25, "name": "p", "time": 1622447562.2994788, "is_keypad": false}
# {"event_type": "up", "scan_code": 25, "name": "p", "time": 1622447562.431007, "is_keypad": false}
# ...

# Replay events
python -m keyboard < events.txt
```

## Known limitations:

- Events generated under Windows don't report device id (`event.device == None`). [#21](https://github.com/boppreh/keyboard/issues/21)
- Media keys on Linux may appear nameless (scan-code only) or not at all. [#20](https://github.com/boppreh/keyboard/issues/20)
- Key suppression/blocking only available on Windows. [#22](https://github.com/boppreh/keyboard/issues/22)
- To avoid depending on X, the Linux parts reads raw device files (`/dev/input/input*`) but this requires root.
- Other applications, such as some games, may register hooks that swallow all key events. In this case `keyboard` will be unable to report events.
- This program makes no attempt to hide itself, so don't use it for keyloggers or online gaming bots. Be responsible.
- SSH connections forward only the text typed, not keyboard events. Therefore if you connect to a server or Raspberry PI that is running `keyboard` via SSH, the server will not detect your key events.

## Common patterns and mistakes

### Preventing the program from closing

```py
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
```

### 等待一次按键

```py
import keyboard

# 别这样！这将使用100%的CPU，直到你按下键。
#
#while not keyboard.is_pressed('space'):
#    continue
#print('空格按下, 继续...')

# Do this instead
keyboard.wait('space')
print('空格按下, 继续...')
```

### 反复等待按键

```py
import keyboard

# 别这样！
#
#while True:
#    if keyboard.is_pressed('space'):
#        print('空格按下！')
#
# 这将使用100%的CPU和打印消息多次。

# 改为这样做
while True:
    keyboard.wait('space')
    print('space was pressed! Waiting on it again...')

# 或者这个
keyboard.add_hotkey('space', lambda: print('space was pressed!'))
keyboard.wait()
```

### 事件发生时调用代码

```py
import keyboard

# Don't do this! This will call `print('space')` immediately then fail when the key is actually pressed.
#keyboard.add_hotkey('space', print('space was pressed'))

# Do this instead
keyboard.add_hotkey('space', lambda: print('space was pressed'))

# or this
def on_space():
    print('space was pressed')
keyboard.add_hotkey('space', on_space)
```

### '按任意键继续'

```py
# Don't do this! The `keyboard` module is meant for global events, even when your program is not in focus.
#import keyboard
#print('Press any key to continue...')
#keyboard.get_event()

# Do this instead
input('Press enter to continue...')

# Or one of the suggestions from here
# https://stackoverflow.com/questions/983354/how-to-make-a-script-wait-for-a-pressed-key
```



# API
#### Table of Contents

- [keyboard.**KEY\_DOWN按键按下**](#keyboard.KEY_DOWN)
- [keyboard.**KEY\_UP按键上升**](#keyboard.KEY_UP)
- [keyboard.**KeyboardEvent键盘事件**](#keyboard.KeyboardEvent)
- [keyboard.**all\_modifiers所有修饰符**](#keyboard.all_modifiers)
- [keyboard.**sided\_modifiers侧边修饰符**](#keyboard.sided_modifiers)
- [keyboard.**version版本**](#keyboard.version)
- [keyboard.**is\_modifier是否修饰符**](#keyboard.is_modifier)
- [keyboard.**key\_to\_scan\_codes按键到扫描 代码**](#keyboard.key_to_scan_codes)
- [keyboard.**parse\_hotkey解析热键**](#keyboard.parse_hotkey)
- [keyboard.**send发送**](#keyboard.send) *(aliases: `press_and_release`)*
- [keyboard.**press按**](#keyboard.press)
- [keyboard.**release释放**](#keyboard.release)
- [keyboard.**is\_pressed是否按下**](#keyboard.is_pressed)
- [keyboard.**call\_later稍后执行**](#keyboard.call_later)
- [keyboard.**hook钩子**](#keyboard.hook)
- [keyboard.**on\_press按下**](#keyboard.on_press)
- [keyboard.**on\_release释放**](#keyboard.on_release)
- [keyboard.**hook\_key钩子 键**](#keyboard.hook_key)
- [keyboard.**on\_press\_key按键按下回调**](#keyboard.on_press_key)
- [keyboard.**on\_release\_key按键释放回调**](#keyboard.on_release_key)
- [keyboard.**unhook脱钩**](#keyboard.unhook) *(aliases: `unblock_key`, `unhook_key`, `unremap_key`)*
- [keyboard.**unhook\_all脱钩所有**](#keyboard.unhook_all)
- [keyboard.**block\_key拦截键**](#keyboard.block_key)
- [keyboard.**remap\_key重新映射键**](#keyboard.remap_key)
- [keyboard.**parse\_hotkey\_combinations解析热键组合**](#keyboard.parse_hotkey_combinations)
- [keyboard.**add\_hotkey新增热键**](#keyboard.add_hotkey) *(aliases: `register_hotkey`)*
- [keyboard.**remove\_hotkey移除热键**](#keyboard.remove_hotkey) *(aliases: `clear_hotkey`, `unregister_hotkey`, `unremap_hotkey`)*
- [keyboard.**unhook\_all\_hotkeys解开所有热键**](#keyboard.unhook_all_hotkeys) *(aliases: `clear_all_hotkeys`, `remove_all_hotkeys`, `unregister_all_hotkeys`)*
- [keyboard.**remap\_hotkey重新映射热键**](#keyboard.remap_hotkey)
- [keyboard.**stash\_state隐藏状态**](#keyboard.stash_state)
- [keyboard.**restore\_state还原状态**](#keyboard.restore_state)
- [keyboard.**restore\_modifiers还原修饰键**](#keyboard.restore_modifiers)
- [keyboard.**write写入**](#keyboard.write)
- [keyboard.**wait等待**](#keyboard.wait)
- [keyboard.**get\_hotkey\_name读取热键名称**](#keyboard.get_hotkey_name)
- [keyboard.**read\_event读取事件**](#keyboard.read_event)
- [keyboard.**read\_key读取键**](#keyboard.read_key)
- [keyboard.**read\_hotkey读取热键**](#keyboard.read_hotkey)
- [keyboard.**get\_typed\_strings获取输入字符串**](#keyboard.get_typed_strings)
- [keyboard.**start\_recording开始录制**](#keyboard.start_recording)
- [keyboard.**stop\_recording停止录制**](#keyboard.stop_recording)
- [keyboard.**record记录**](#keyboard.record)
- [keyboard.**play播放**](#keyboard.play) *(aliases: `replay`)*
- [keyboard.**add\_word\_listener新增文字监听**](#keyboard.add_word_listener) *(aliases: `register_word_listener`)*
- [keyboard.**remove\_word\_listener移除文字监听**](#keyboard.remove_word_listener) *(aliases: `remove_abbreviation`)*
- [keyboard.**add\_abbreviation新增缩写**](#keyboard.add_abbreviation) *(aliases: `register_abbreviation`)*
- [keyboard.**normalize\_name规范化名称**](#keyboard.normalize_name)


<a name="keyboard.KEY_DOWN"/>

## keyboard.**KEY\_DOWN**
```py
= 'down'
```

<a name="keyboard.KEY_UP"/>

## keyboard.**KEY\_UP**
```py
= 'up'
```

<a name="keyboard.KeyboardEvent"/>

## class keyboard.**KeyboardEvent**




<a name="KeyboardEvent.device"/>

### KeyboardEvent.**device**


<a name="KeyboardEvent.event_type"/>

### KeyboardEvent.**event\_type**


<a name="KeyboardEvent.is_keypad"/>

### KeyboardEvent.**is\_keypad**


<a name="KeyboardEvent.modifiers"/>

### KeyboardEvent.**modifiers**


<a name="KeyboardEvent.name"/>

### KeyboardEvent.**name**


<a name="KeyboardEvent.scan_code"/>

### KeyboardEvent.**scan\_code**


<a name="KeyboardEvent.time"/>

### KeyboardEvent.**time**


<a name="KeyboardEvent.to_json"/>

### KeyboardEvent.**to\_json**(self, ensure\_ascii=False)

[\[source\]](https://github.com/boppreh/keyboard/blob/master/keyboard/_keyboard_event.py#L34)






<a name="keyboard.all_modifiers"/>

## keyboard.**all\_modifiers**
```py
= {'alt', 'alt gr', 'ctrl', 'left alt', 'left ctrl', 'left shift', 'left windows', 'right alt', 'right ctrl', 'right shift', 'right windows', 'shift', 'windows'}
```

<a name="keyboard.sided_modifiers"/>

## keyboard.**sided\_modifiers**
```py
= {'alt', 'ctrl', 'shift', 'windows'}
```

<a name="keyboard.version"/>

## keyboard.**version**
```py
= '0.13.5'
```

<a name="keyboard.is_modifier"/>

## keyboard.**is\_modifier**(key)

[\[source\]](https://github.com/boppreh/keyboard/blob/master/keyboard/__init__.py#L242)


Returns True if `key` is a scan code or name of a modifier key.



<a name="keyboard.key_to_scan_codes"/>

## keyboard.**key\_to\_scan\_codes**(key, error\_if\_missing=True)

[\[source\]](https://github.com/boppreh/keyboard/blob/master/keyboard/__init__.py#L405)


Returns a list of scan codes associated with this key (name or scan code).



<a name="keyboard.parse_hotkey"/>

## keyboard.**parse\_hotkey**(hotkey)

[\[source\]](https://github.com/boppreh/keyboard/blob/master/keyboard/__init__.py#L435)


Parses a user-provided hotkey into nested tuples representing the
parsed structure, with the bottom values being lists of scan codes.
Also accepts raw scan codes, which are then wrapped in the required
number of nestings.

Example:

```py

parse_hotkey("alt+shift+a, alt+b, c")
#    Keys:    ^~^ ^~~~^ ^  ^~^ ^  ^
#    Steps:   ^~~~~~~~~~^  ^~~~^  ^

# ((alt_codes, shift_codes, a_codes), (alt_codes, b_codes), (c_codes,))
```



<a name="keyboard.send"/>

## keyboard.**send**(hotkey, do\_press=True, do\_release=True)

[\[source\]](https://github.com/boppreh/keyboard/blob/master/keyboard/__init__.py#L468)


Sends OS events that perform the given *hotkey* hotkey.

- `hotkey` can be either a scan code (e.g. 57 for space), single key
(e.g. 'space') or multi-key, multi-step hotkey (e.g. 'alt+F4, enter').
- `do_press` if true then press events are sent. Defaults to True.
- `do_release` if true then release events are sent. Defaults to True.

```py

send(57)
send('ctrl+alt+del')
send('alt+F4, enter')
send('shift+s')
```

Note: keys are released in the opposite order they were pressed.



<a name="keyboard.press"/>

## keyboard.**press**(hotkey)

[\[source\]](https://github.com/boppreh/keyboard/blob/master/keyboard/__init__.py#L501)

Presses and holds down a hotkey (see [`send`](#keyboard.send)). 


<a name="keyboard.release"/>

## keyboard.**release**(hotkey)

[\[source\]](https://github.com/boppreh/keyboard/blob/master/keyboard/__init__.py#L505)

Releases a hotkey (see [`send`](#keyboard.send)). 


<a name="keyboard.is_pressed"/>

## keyboard.**is\_pressed**(hotkey)

[\[source\]](https://github.com/boppreh/keyboard/blob/master/keyboard/__init__.py#L509)


Returns True if the key is pressed.

```py

is_pressed(57) #-> True
is_pressed('space') #-> True
is_pressed('ctrl+space') #-> True
```



<a name="keyboard.call_later"/>

## keyboard.**call\_later**(fn, args=(), delay=0.001)

[\[source\]](https://github.com/boppreh/keyboard/blob/master/keyboard/__init__.py#L536)


Calls the provided function in a new thread after waiting some time.
Useful for giving the system some time to process an event, without blocking
the current execution flow.



<a name="keyboard.hook"/>

## keyboard.**hook**(callback, suppress=False, on\_remove=&lt;lambda&gt;)

[\[source\]](https://github.com/boppreh/keyboard/blob/master/keyboard/__init__.py#L546)


Installs a global listener on all available keyboards, invoking `callback`
each time a key is pressed or released.

The event passed to the callback is of type `keyboard.KeyboardEvent`,
with the following attributes:

- `name`: an Unicode representation of the character (e.g. "&") or
description (e.g.  "space"). The name is always lower-case.
- `scan_code`: number representing the physical key, e.g. 55.
- `time`: timestamp of the time the event occurred, with as much precision
as given by the OS.

Returns the given callback for easier development.



<a name="keyboard.on_press"/>

## keyboard.**on\_press**(callback, suppress=False)

[\[source\]](https://github.com/boppreh/keyboard/blob/master/keyboard/__init__.py#L577)


Invokes `callback` for every KEY_DOWN event. For details see [`hook`](#keyboard.hook).



<a name="keyboard.on_release"/>

## keyboard.**on\_release**(callback, suppress=False)

[\[source\]](https://github.com/boppreh/keyboard/blob/master/keyboard/__init__.py#L583)


Invokes `callback` for every KEY_UP event. For details see [`hook`](#keyboard.hook).



<a name="keyboard.hook_key"/>

## keyboard.**hook\_key**(key, callback, suppress=False)

[\[source\]](https://github.com/boppreh/keyboard/blob/master/keyboard/__init__.py#L589)


Hooks key up and key down events for a single key. Returns the event handler
created. To remove a hooked key use [`unhook_key(key)`](#keyboard.unhook_key) or
[`unhook_key(handler)`](#keyboard.unhook_key).

Note: this function shares state with hotkeys, so [`clear_all_hotkeys`](#keyboard.clear_all_hotkeys)
affects it as well.



<a name="keyboard.on_press_key"/>

## keyboard.**on\_press\_key**(key, callback, suppress=False)

[\[source\]](https://github.com/boppreh/keyboard/blob/master/keyboard/__init__.py#L613)


Invokes `callback` for KEY_DOWN event related to the given key. For details see [`hook`](#keyboard.hook).



<a name="keyboard.on_release_key"/>

## keyboard.**on\_release\_key**(key, callback, suppress=False)

[\[source\]](https://github.com/boppreh/keyboard/blob/master/keyboard/__init__.py#L619)


Invokes `callback` for KEY_UP event related to the given key. For details see [`hook`](#keyboard.hook).



<a name="keyboard.unhook"/>

## keyboard.**unhook**(remove)

[\[source\]](https://github.com/boppreh/keyboard/blob/master/keyboard/__init__.py#L625)


Removes a previously added hook, either by callback or by the return value
of [`hook`](#keyboard.hook).



<a name="keyboard.unhook_all"/>

## keyboard.**unhook\_all**()

[\[source\]](https://github.com/boppreh/keyboard/blob/master/keyboard/__init__.py#L633)


Removes all keyboard hooks in use, including hotkeys, abbreviations, word
listeners, [`record`](#keyboard.record)ers and [`wait`](#keyboard.wait)s.



<a name="keyboard.block_key"/>

## keyboard.**block\_key**(key)

[\[source\]](https://github.com/boppreh/keyboard/blob/master/keyboard/__init__.py#L645)


Suppresses all key events of the given key, regardless of modifiers.



<a name="keyboard.remap_key"/>

## keyboard.**remap\_key**(src, dst)

[\[source\]](https://github.com/boppreh/keyboard/blob/master/keyboard/__init__.py#L652)


Whenever the key `src` is pressed or released, regardless of modifiers,
press or release the hotkey `dst` instead.



<a name="keyboard.parse_hotkey_combinations"/>

## keyboard.**parse\_hotkey\_combinations**(hotkey)

[\[source\]](https://github.com/boppreh/keyboard/blob/master/keyboard/__init__.py#L666)


Parses a user-provided hotkey. Differently from [`parse_hotkey`](#keyboard.parse_hotkey),
instead of each step being a list of the different scan codes for each key,
each step is a list of all possible combinations of those scan codes.



<a name="keyboard.add_hotkey"/>

## keyboard.**add\_hotkey**(hotkey, callback, args=(), suppress=False, timeout=1, trigger\_on\_release=False)

[\[source\]](https://github.com/boppreh/keyboard/blob/master/keyboard/__init__.py#L706)


Invokes a callback every time a hotkey is pressed. The hotkey must
be in the format `ctrl+shift+a, s`. This would trigger when the user holds
ctrl, shift and "a" at once, releases, and then presses "s". To represent
literal commas, pluses, and spaces, use their names ('comma', 'plus',
'space').

- `args` is an optional list of arguments to passed to the callback during
each invocation.
- `suppress` defines if successful triggers should block the keys from being
sent to other programs.
- `timeout` is the amount of seconds allowed to pass between key presses.
- `trigger_on_release` if true, the callback is invoked on key release instead
of key press.

The event handler function is returned. To remove a hotkey call
[`remove_hotkey(hotkey)`](#keyboard.remove_hotkey) or [`remove_hotkey(handler)`](#keyboard.remove_hotkey).
before the hotkey state is reset.

Note: hotkeys are activated when the last key is *pressed*, not released.
Note: the callback is executed in a separate thread, asynchronously. For an
example of how to use a callback synchronously, see [`wait`](#keyboard.wait).

Examples:

```py

# Different but equivalent ways to listen for a spacebar key press.
add_hotkey(' ', print, args=['space was pressed'])
add_hotkey('space', print, args=['space was pressed'])
add_hotkey('Space', print, args=['space was pressed'])
# Here 57 represents the keyboard code for spacebar; so you will be
# pressing 'spacebar', not '57' to activate the print function.
add_hotkey(57, print, args=['space was pressed'])

add_hotkey('ctrl+q', quit)
add_hotkey('ctrl+alt+enter, space', some_callback)
```



<a name="keyboard.remove_hotkey"/>

## keyboard.**remove\_hotkey**(hotkey\_or\_callback)

[\[source\]](https://github.com/boppreh/keyboard/blob/master/keyboard/__init__.py#L852)


Removes a previously hooked hotkey. Must be called with the value returned
by [`add_hotkey`](#keyboard.add_hotkey).



<a name="keyboard.unhook_all_hotkeys"/>

## keyboard.**unhook\_all\_hotkeys**()

[\[source\]](https://github.com/boppreh/keyboard/blob/master/keyboard/__init__.py#L860)


Removes all keyboard hotkeys in use, including abbreviations, word listeners,
[`record`](#keyboard.record)ers and [`wait`](#keyboard.wait)s.



<a name="keyboard.remap_hotkey"/>

## keyboard.**remap\_hotkey**(src, dst, suppress=True, trigger\_on\_release=False)

[\[source\]](https://github.com/boppreh/keyboard/blob/master/keyboard/__init__.py#L871)


Whenever the hotkey `src` is pressed, suppress it and send
`dst` instead.

Example:

```py

remap('alt+w', 'ctrl+up')
```



<a name="keyboard.stash_state"/>

## keyboard.**stash\_state**()

[\[source\]](https://github.com/boppreh/keyboard/blob/master/keyboard/__init__.py#L891)


Builds a list of all currently pressed scan codes, releases them and returns
the list. Pairs well with [`restore_state`](#keyboard.restore_state) and [`restore_modifiers`](#keyboard.restore_modifiers).



<a name="keyboard.restore_state"/>

## keyboard.**restore\_state**(scan\_codes)

[\[source\]](https://github.com/boppreh/keyboard/blob/master/keyboard/__init__.py#L903)


Given a list of scan_codes ensures these keys, and only these keys, are
pressed. Pairs well with [`stash_state`](#keyboard.stash_state), alternative to [`restore_modifiers`](#keyboard.restore_modifiers).



<a name="keyboard.restore_modifiers"/>

## keyboard.**restore\_modifiers**(scan\_codes)

[\[source\]](https://github.com/boppreh/keyboard/blob/master/keyboard/__init__.py#L920)


Like [`restore_state`](#keyboard.restore_state), but only restores modifier keys.



<a name="keyboard.write"/>

## keyboard.**write**(text, delay=0, restore\_state\_after=True, exact=None)

[\[source\]](https://github.com/boppreh/keyboard/blob/master/keyboard/__init__.py#L926)


Sends artificial keyboard events to the OS, simulating the typing of a given
text. Characters not available on the keyboard are typed as explicit unicode
characters using OS-specific functionality, such as alt+codepoint.

To ensure text integrity, all currently pressed keys are released before
the text is typed, and modifiers are restored afterwards.

- `delay` is the number of seconds to wait between keypresses, defaults to
no delay.
- `restore_state_after` can be used to restore the state of pressed keys
after the text is typed, i.e. presses the keys that were released at the
beginning. Defaults to True.
- `exact` forces typing all characters as explicit unicode (e.g.
alt+codepoint or special events). If None, uses platform-specific suggested
value.



<a name="keyboard.wait"/>

## keyboard.**wait**(hotkey=None, suppress=False, trigger\_on\_release=False)

[\[source\]](https://github.com/boppreh/keyboard/blob/master/keyboard/__init__.py#L981)


Blocks the program execution until the given hotkey is pressed or,
if given no parameters, blocks forever.



<a name="keyboard.get_hotkey_name"/>

## keyboard.**get\_hotkey\_name**(names=None)

[\[source\]](https://github.com/boppreh/keyboard/blob/master/keyboard/__init__.py#L995)


Returns a string representation of hotkey from the given key names, or
the currently pressed keys if not given.  This function:

- normalizes names;
- removes "left" and "right" prefixes;
- replaces the "+" key name with "plus" to avoid ambiguity;
- puts modifier keys first, in a standardized order;
- sort remaining keys;
- finally, joins everything with "+".

Example:

```py

get_hotkey_name(['+', 'left ctrl', 'shift'])
# "ctrl+shift+plus"
```



<a name="keyboard.read_event"/>

## keyboard.**read\_event**(suppress=False)

[\[source\]](https://github.com/boppreh/keyboard/blob/master/keyboard/__init__.py#L1026)


Blocks until a keyboard event happens, then returns that event.



<a name="keyboard.read_key"/>

## keyboard.**read\_key**(suppress=False)

[\[source\]](https://github.com/boppreh/keyboard/blob/master/keyboard/__init__.py#L1037)


Blocks until a keyboard event happens, then returns that event's name or,
if missing, its scan code.



<a name="keyboard.read_hotkey"/>

## keyboard.**read\_hotkey**(suppress=True)

[\[source\]](https://github.com/boppreh/keyboard/blob/master/keyboard/__init__.py#L1045)


Similar to [`read_key()`](#keyboard.read_key), but blocks until the user presses and releases a
hotkey (or single key), then returns a string representing the hotkey
pressed.

Example:

```py

read_hotkey()
# "ctrl+shift+p"
```



<a name="keyboard.get_typed_strings"/>

## keyboard.**get\_typed\_strings**(events, allow\_backspace=True)

[\[source\]](https://github.com/boppreh/keyboard/blob/master/keyboard/__init__.py#L1067)


Given a sequence of events, tries to deduce what strings were typed.
Strings are separated when a non-textual key is pressed (such as tab or
enter). Characters are converted to uppercase according to shift and
capslock status. If `allow_backspace` is True, backspaces remove the last
character typed.

This function is a generator, so you can pass an infinite stream of events
and convert them to strings in real time.

Note this functions is merely an heuristic. Windows for example keeps per-
process keyboard state such as keyboard layout, and this information is not
available for our hooks.

```py

get_type_strings(record()) #-> ['This is what', 'I recorded', '']
```



<a name="keyboard.start_recording"/>

## keyboard.**start\_recording**(recorded\_events\_queue=None)

[\[source\]](https://github.com/boppreh/keyboard/blob/master/keyboard/__init__.py#L1114)


Starts recording all keyboard events into a global variable, or the given
queue if any. Returns the queue of events and the hooked function.

Use [`stop_recording()`](#keyboard.stop_recording) or [`unhook(hooked_function)`](#keyboard.unhook) to stop.



<a name="keyboard.stop_recording"/>

## keyboard.**stop\_recording**()

[\[source\]](https://github.com/boppreh/keyboard/blob/master/keyboard/__init__.py#L1126)


Stops the global recording of events and returns a list of the events
captured.



<a name="keyboard.record"/>

## keyboard.**record**(until=&#x27;escape&#x27;, suppress=False, trigger\_on\_release=False)

[\[source\]](https://github.com/boppreh/keyboard/blob/master/keyboard/__init__.py#L1138)


Records all keyboard events from all keyboards until the user presses the
given hotkey. Then returns the list of events recorded, of type
`keyboard.KeyboardEvent`. Pairs well with
[`play(events)`](#keyboard.play).

Note: this is a blocking function.
Note: for more details on the keyboard hook and events see [`hook`](#keyboard.hook).



<a name="keyboard.play"/>

## keyboard.**play**(events, speed\_factor=1.0)

[\[source\]](https://github.com/boppreh/keyboard/blob/master/keyboard/__init__.py#L1152)


Plays a sequence of recorded events, maintaining the relative time
intervals. If speed_factor is <= 0 then the actions are replayed as fast
as the OS allows. Pairs well with [`record()`](#keyboard.record).

Note: the current keyboard state is cleared at the beginning and restored at
the end of the function.



<a name="keyboard.add_word_listener"/>

## keyboard.**add\_word\_listener**(word, callback, triggers=[&#x27;space&#x27;], match\_suffix=False, timeout=2)

[\[source\]](https://github.com/boppreh/keyboard/blob/master/keyboard/__init__.py#L1176)


Invokes a callback every time a sequence of characters is typed (e.g. 'pet')
and followed by a trigger key (e.g. space). Modifiers (e.g. alt, ctrl,
shift) are ignored.

- `word` the typed text to be matched. E.g. 'pet'.
- `callback` is an argument-less function to be invoked each time the word
is typed.
- `triggers` is the list of keys that will cause a match to be checked. If
the user presses some key that is not a character (len>1) and not in
triggers, the characters so far will be discarded. By default the trigger
is only `space`.
- `match_suffix` defines if endings of words should also be checked instead
of only whole words. E.g. if true, typing 'carpet'+space will trigger the
listener for 'pet'. Defaults to false, only whole words are checked.
- `timeout` is the maximum number of seconds between typed characters before
the current word is discarded. Defaults to 2 seconds.

Returns the event handler created. To remove a word listener use
[`remove_word_listener(word)`](#keyboard.remove_word_listener) or [`remove_word_listener(handler)`](#keyboard.remove_word_listener).

Note: all actions are performed on key down. Key up events are ignored.
Note: word matches are **case sensitive**.



<a name="keyboard.remove_word_listener"/>

## keyboard.**remove\_word\_listener**(word\_or\_handler)

[\[source\]](https://github.com/boppreh/keyboard/blob/master/keyboard/__init__.py#L1232)


Removes a previously registered word listener. Accepts either the word used
during registration (exact string) or the event handler returned by the
[`add_word_listener`](#keyboard.add_word_listener) or [`add_abbreviation`](#keyboard.add_abbreviation) functions.



<a name="keyboard.add_abbreviation"/>

## keyboard.**add\_abbreviation**(source\_text, replacement\_text, match\_suffix=False, timeout=2)

[\[source\]](https://github.com/boppreh/keyboard/blob/master/keyboard/__init__.py#L1240)


Registers a hotkey that replaces one typed text with another. For example

```py

add_abbreviation('tm', u'™')
```

Replaces every "tm" followed by a space with a ™ symbol (and no space). The
replacement is done by sending backspace events.

- `match_suffix` defines if endings of words should also be checked instead
of only whole words. E.g. if true, typing 'carpet'+space will trigger the
listener for 'pet'. Defaults to false, only whole words are checked.
- `timeout` is the maximum number of seconds between typed characters before
the current word is discarded. Defaults to 2 seconds.

For more details see [`add_word_listener`](#keyboard.add_word_listener).



<a name="keyboard.normalize_name"/>

## keyboard.**normalize\_name**(name)

[\[source\]](https://github.com/boppreh/keyboard/blob/master/keyboard/_canonical_names.py#L1233)


Given a key name (e.g. "LEFT CONTROL"), clean up the string and convert to
the canonical representation (e.g. "left ctrl") if one is known.


