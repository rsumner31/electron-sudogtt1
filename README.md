## Electron subprocess with administrative privileges

Run a subprocess with administrative privileges, prompting the user with a graphical OS dialog if necessary. Useful for background subprocesse which run native Electron apps that need sudo.

- `Windows`, uses native ```User Account Control (UAC)``` (no ```PowerShell``` required)
- `OS X`, uses bundled applet (inspired by  [Joran Dirk Greef](https://github.com/jorangreef))
- `Linux`, uses system pkexec or gksudo (system or bundled).

<img height="150px" src="./assets/win32.png">
<img height="150px" src="./assets/osx.png">
<img height="150px" src="./assets/linux.png">

- `Linux`, we try to use system pkexec, gksudo or kdesudo, if it not found then use bundled gksu.

    <img src="./linux/sample1.png" width="330px">

    <img src="./linux/sample2.png" width="330px">

## Installation
```
npm install electron-sudo
```

## Usage
**Note: Your command should not start with the ```sudo``` prefix.**

### Version 4.0.*
```js
import Sudoer from 'electron-sudo';
let options = {name: 'electron sudo application'},
  sudoer = new Sudoer(options);
/*
 Spawn subprocess behavior
*/
let cp = await sudoer.spawn(
  'echo', ['$PARAM'], {env: {PARAM: 'VALUE'}}
);
cp.on('close', () => {
  // cp.output.stdout (Buffer)
  // cp.output.stderr (Buffer)
});

/*
 Exec subprocess behavior
*/
let result = await sudoer.exec(
  'echo $PARAM', {env: {PARAM: 'VALUE'}}
);
// result is Buffer with mixed (both stdout and stderr) output
```

### Version 3.0.* (deprecated)
```js
var sudo = require('electron-sudo');
var options = {
  name: 'Your application name',
  icns: '/path/to/icns/file' // (optional, only for MacOS),
  process: {
    options: {
      // Can use custom environment variables for your privileged subprocess
      env: {'VAR': 'VALUE'}
      // ... and all other subprocess options described here
      // https://nodejs.org/api/child_process.html#child_process_child_process_exec_command_options_callback
    },
    on: function(ps) {
      ps.stdout.on('data', function(data) {});
      setTimeout(function() {
        ps.kill()
      }.bind(ps), 50000);
    }
  }
};
sudo.exec('echo hello', options, function(error) {});
```

`electron-sudo` will use `process.title` as `options.name` if `options.name` is not provided. `options.name` must be alphanumeric only (spaces are supported) and at most 70 characters.


## Behavior
- ```OS X```, `electron-sudo` should behave just like the `sudo` command in the shell. If your command does not work with the `sudo` command in the shell (perhaps because it uses `>` redirection to a restricted file), then it will not work with `electron-sudo`. However, it is still possible to use electron-sudo to get a privileged shell.

*Please note that Linux support is currently in beta and requires more testing across Linux distributions.*

- ```Linux```, `electron-sudo` will use embedded `gksu` to show the password prompt and run your command. Where possible, `electron-sudo` will try and get these to mimic `sudo` as much as possible (for example by preserving environment), but your command should not rely on any environment variables or relative paths, in order to work correctly. Depending on which binary is used, and due to the limitations of some binaries, the name of your program or the command itself may be displayed to your user.

Just as you should never use `sudo` to launch any graphical applications, you should never use `electron-sudo` to launch any graphical applications. Doing so could cause files in your home directory to become owned by root. `electron-sudo` is explicitly designed to launch non-graphical terminal commands. For more information, [read this post](http://www.psychocats.net/ubuntu/graphicalsudo).

## Platform-specific sources

- [Applet [OSx]](https://github.com/automation-stack/electron-sudo/tree/master/darwin/applet.app/Contents)
- [Elevate.exe sources [Win32/64]](https://github.com/automation-stack/electron-sudo/tree/master/win32/src)
- [GKsu sources [Linux]](https://github.com/automation-stack/electron-sudo/tree/master/linux)

If you don't trust binaries bundled in NPM package you can manually build tools and use them instead.
