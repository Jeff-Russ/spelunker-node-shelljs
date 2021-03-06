# spelunker()

[On GitHub](https://github.com/Jeff-Russ/spelunker-node-shell)


## Rapid Shell Executioner for Node.js  

`spelunker()` sends your guy into the shell and brings back the goods quicker than you going in and out for each command. Spelunker allows you to build up a list of shell commands to be executed all and once and store the results of each separately to an object you provide.  

Without having to start a new sub-shell for each command. spelunker goes in and gathers all results in shell variables and echoes them all out as JS syntax as fodder for eval() to execute and add to your object (first arg). spelunker only makes one trip into the shell/cave to do all of it's gathering of data so it's faster!  

# WARNING

This little function uses `exec()` which executes raw strings as JavaScript without any sort of checking. To make thing even more of a risk, it's executing information handed back by the system's shell, effectively bursting the secure bubble that shields you Node environment from the OS. Spelunker was created for making an installer script run only by a developer on the a development machine and is not intended to live on a server or in a Node.js application.  

## Installation  

In terminal, from anywhere in your project:  

```bash
$ curl -O https://raw.githubusercontent.com/Jeff-Russ/spelunker-node-shell/master/spelunker.js
```

I also recommend downloading the man page, which is directly runnable in any location with `./spelunker.man`  

```bash
$ curl -O https://raw.githubusercontent.com/Jeff-Russ/spelunker-node-shell/master/spelunker.man
$ ./spelunker.man  # displays manual
```

If you get access denied when trying to run either, use `chmod u+x ./filename`.  

Then in your JS file: 

```javascript
require('./path/to/spelunker.js').globalize();
```

although the `.js` part isn't needed. The `.globalize()` call makes the `spelunker()` method directly available but you could also do:  

```javascript
var x = require('./path/to/spelunker.js');

x.spelunker(your args);
```

It's annoying but common.  

## The Function Definition

The first argument is an object you fill with commands you fill with commands before calling `spelunker()`. It's see below as `commands`. Each property name will the the property name created in a destination object, which will hold the captured outputs of each command from the commands object.  

The destination object, seen below as `info`, can be provided as the second argument and `spelunker()` will write to it. Any property names matching those in the commands object will be overwritten with the new output. The rest of the properties will remain untouched. If you don't provide a second argument, `spelunker()` will spit out a new one. Either way there will be a return, but whether or not it's a newly created object depends on whether you provided one.  

__NOTE__ that `spelunker` will add a `.iter(fn)` method to the commands object so you should avoid this property name.  


## Example Use: 

```javascript

info = {
  caller_dir: pwd().stdout,
  to_dir: "./to",
  from_dir: "./from",
  def_from_dir: __dirname + "/appgen-templates/default",
  conf_file: 'appgen-config.sh'
};

commands = {
  to_dir: "cd " + info.to_dir + "; pwd",
  from_dir: "cd " + info.from_dir + "; pwd",
  sys_user: 'id -F || id -un || whoami || git config user.name || \'\'',
  git_email: 'git config user.email || \'\'',
  author: 'git config user.name || id -F || id -un || whoami || \'\'',
  to_dir_exists: "test -d " + info.to_dir + " && echo true || echo false",
  to_dir_empty: "test \"$(ls -A " + info.to_dir + ")\" && echo false || echo true",
  to_dir_has_git: "test -d " + info.to_dir + "/.git && echo true || echo false",
  to_dir_has_json: "test -f " + info.to_dir + "/package.json && echo true || echo false",
  to_dir_readme: "ls \"" + info.to_dir + "\" | grep -i readme || echo false",
  from_dir_exists: "test -d " + info.from_dir + " && echo true || echo false",
  from_dir_config: "test -f " + info.from_dir + "/" + info.conf_file + " && echo true || echo false"
};

spelunker(commands, info);
console.log(info);
```

this prints out:  

```bash
{ caller_dir: '/Users/Jeff/bin',
  to_dir: '/Users/Jeff/bin/to',
  from_dir: '/Users/Jeff/bin/from',
  def_from_dir: '/Users/Jeff/bin/appgen-templates/default',
  conf_file: 'appgen-config.sh',
  sys_user: 'Jeffrey Russ',
  git_email: 'jeffreylynnruss@gmail.com',
  author: 'Jeff-Russ',
  to_dir_exists: true,
  to_dir_empty: false,
  to_dir_has_git: true,
  to_dir_has_json: false,
  to_dir_readme: 'Readme.rdoc',
  from_dir_exists: true,
  from_dir_config: true }
```
Notice you're not seeing `'true'` but `true`. Spelunker will post-process the results object to replace any string representing a boolean to a boolean. ~~It also does the same with integers and floats.~~  

## Another Way

If you wanted to not have two object and just write back to the same object that has the command you can provide the same argument twice without any issue:  
```javascript
spelunker(shellvars, shellvars);
```

___YOU MUST BE SURE__ that the object only contains valid shell commands and __nothing else__. After you've run this it's all over, don't put that spelunk that cave again, unless you want some wild things happening in your shell!!  

In any case, now that your output object entered the cave as a command object it will have the `iter()` method added, which might be a nice bonus.  It takes a single argument: a function that is called for each property and given the property name as it's argument.  Here is it in use:  

```javascript
info.iter(function(prop){
  if (info[prop] == true) console.log(prop);
});
```

## Under the Hood 

Here is what the above example sends to the shell:  

```bash
to_dir=info.to_dir" = ""'$(cd ./to; pwd)'";
from_dir=info.from_dir" = ""'$(cd ./from; pwd)'";
# ... etc ...
echo "$to_dir""$from_dir""$sys_user""$git_email""$author"# ... etc ...
```

As you can see, it's creating shell variables and then echoing them all back out. Here is the echoed output:  

```javascript
info.to_dir = '/Users/Jeff/bin/to'
info.from_dir = '/Users/Jeff/bin/from'
info.sys_user = 'Jeffrey Russ'
info.git_email = 'jeffreylynnruss@gmail.com'
info.author = 'Jeff-Russ'
info.to_dir_exists = true
info.to_dir_empty = false
info.to_dir_has_git = true
info.to_dir_has_json = false
info.to_dir_readme = 'Readme.rdoc'
info.from_dir_exists = true
info.from_dir_config = true
```

Look familiar? It's just Javascript. spelunker then runs this with `eval()` which saves to the object!  

