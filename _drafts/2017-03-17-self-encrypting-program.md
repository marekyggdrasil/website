---
layout: post
title: Self encrypting program
tags: [Ideas, Fun, Python]
---

## Motivation

I carry around a lot of homemade software, I have it on my USB stick, I send it to myself via email or keep it in my Google Drive or Amazon Drive.

Most of them are little command line tools I prepare for various purposes such as sending or reading messages, logging information, synchronizing it etc. They often require to hold credentials, private PGP keys and other things you wouldn't like to be found by a third party.

This kind of tool rarely requires lot of storage space, but data it saves is highly confidential, also those are little projects, can fit in single file, saving data in the same file (yes! in the source file!) was very tempting thing I wanted to try for a long time!

## Captain's log

Every Starfleet Captain knows that if his ship will be captured by Romulans somewhere in Neutral Zone, if not destroyed they will search it for all possible information about United Federation of Planets, they will also search out computers memory. Captain's log must be stored in encrypted form so that if someone captures the computer it will not be possible to know anything about ship's mission.

### How it works?

### Kernel source code

By kernel I named the main functionality of the application. Kernel is supposed to store it's own data in `state` variable which is pure text.

```python
def run(state) :
    import time
    print '  Welcome to secure Captain\'s log, type help'
    cmd = raw_input('> ')
    while (cmd != 'exit') :
        if cmd == 'help' :
            print '  help   - this screen\n  add    - insert a new note\n  log    - read previous notes\n  exit   - leave starfleet captain\'s log'
        elif cmd == 'add' :
            new = raw_input('< ')
            state += str(time.time()) + ' ' + new + '\n'
        elif cmd == 'log' :
            print 'stardate      message'
            print state
        elif cmd == 'exit' :
            return state
        cmd = raw_input('> ')
```

### Build script source code

The purpose of build script is to initiate the secure application, set the password and encrypt initial data as well as the source code.

### End result

## To do

* mention pure python packages
* mention possibility of combining them into single source using stickytape module
* mention obfuscating

## Security disclaimer

The purpose of this article is purely educational. Software such as one described in this example still holds many potential security vulnerabilities and I do not take any responsibility for consequences of using it.
