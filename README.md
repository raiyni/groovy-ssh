Gradle SSH Plugin
=================

[![Build Status](https://travis-ci.org/int128/gradle-ssh-plugin.png?branch=master)](https://travis-ci.org/int128/gradle-ssh-plugin)

This plugin provides remote command execution and file transfer capabilities.

See also [Example build.gradle](example/build.gradle) and [Change Log](https://github.com/int128/gradle-ssh-plugin/wiki/ChangeLog).


How to use
----------

Add a dependency and apply it in your build.gradle:

```groovy
buildscript {
  repositories {
    mavenCentral()
  }
  dependencies {
    classpath 'org.hidetake:gradle-ssh-plugin:0.1.8'
  }
}

apply plugin: 'ssh'
```


Define remote hosts
-------------------

At first, define remote hosts:

```groovy
remotes {
  web01 {
    host = '192.168.1.101'
    user = 'jenkins'
    identity = file('config/identity.key')
  }
}
```

A remote host instance has following properties:

  * `host` - Hostname or IP address
  * `port` - Port. Default is 22. (Optional)
  * `user` - User name.
  * `password` - Password for password authentication. (Optional)
  * `identity` - Private key for public-key authentication. (Optional)
  * `passphrase` - Passphrase for private key when using public-key authentication. (Optional)

Also remote hosts can be associated with roles, using `role(name)` like:

```groovy
remotes {
  web01 {
    role('webServers')
    host = '192.168.1.101'
    //...
  }
  web02 {
    role('webServers')
    host = '192.168.1.102'
    //...
  }
  web03 {
    role('webServers')
    host = '192.168.1.103'
    //...
  }
}
```

Calling the function `remotes.role()` with roles will return list of remote hosts associated with those roles.


Define a SSH task
-----------------

To define a SSH task, write `task(type: SshTask)` like:

```groovy
task checkWebServer(type: SshTask) {
  session(remotes.web01) {
    def pids = execute('pidof nginx').trim().split(/ /)
    assert pids.length > 1
  }
}

task reloadServers(type: SshTask) {
  session(remotes.role('webServers', 'dbServers')) {
    executeBackground('sudo service httpd reload', pty: true)
  }
}
```


### Task configuration

In the `SshTask` closure, following methods and properties are available:
  * `session(remote)` - Adds a session to the remote host.
  * `session(remotes)` - Adds each session of remote hosts. If a list is given, sessions will be executed in order. Otherwise, order is not defined.
  * `config(key: value)` - Adds an configuration entry. All configurations are given to JSch. This method overwrites entries if same defined in convention.
  * `dryRun` - Dry run flag. If true, performs no action. Default is according to the convention property.
  * `outputLogLevel` - Log level of standard output while command execution. Default is according to the convention property.
  * `errorLogLevel` - Log level of standard error while command execution. Default is according to the convention property.
  * `logger` - (deprecated)

Specification of the closure is defined in the [class SshSpec](src/main/groovy/org/hidetake/gradle/ssh/api/SshSpec.groovy).
Note that the closure will be called in **evaluation** phase on Gradle.


### Session operation

In the `session` closure, following methods and properties are available:
  * `execute(command)` - Executes a command. This method blocks until the command is completed and returns output of the command.
  * `executeSudo(command)` - Executes a command as sudo (prepends sudo -S -p). Used to support sudo commands requiring password. This method blocks until the command is completed and returns output of the command.
  * `executeBackground(command)` - Executes a command in background. Other operations will be performed concurrently.
  * `get(remote, local)` - Fetches a file or directory from remote host.
  * `put(local, remote)` - Sends a file or directory to remote host.
  * `remote` - Remote host of current session. (Read only)

Specification of the closure is defined in the [interface OperationHandler](src/main/groovy/org/hidetake/gradle/ssh/api/OperationHandler.groovy).
Note that the closure will be called in **execution** phase on Gradle.

Above operations accepts option arguments.
For instance, adding `pty: true` makes the channel to request PTY allocation (to execute sudo).


Use SSH in the task
-------------------

To execute SSH in the task, call `sshexec()` method with a closure:

```groovy
task prepareEnvironment {
  doLast {
    def operation = 'reload'
    sshexec {
      session(remotes.role('webServers')) {
        execute("sudo service httpd ${operation}", pty: true)
      }
    }
  }
}
```


Convention properties
---------------------

Global settings can be defined in the `ssh` closure:

```groovy
ssh {
  dryRun = true
  config(StrictHostKeyChecking: 'no')
}
```

Following properties and methods are available:

  * `config(key: value)` - Adds configuration entries. All configurations are passed to JSch.
  * `dryRun` - Dry run flag. If true, performs no action. Default is false.
  * `retryCount` - Retrying count to establish connection. Default is 0 (no retry).
  * `retryWaitSec` - Time in seconds between each retries. Default is 0 (immediately).
  * `outputLogLevel` - Log level of standard output while command execution. Default is `LogLevel.QUIET`.
  * `errorLogLevel` - Log level of standard error while command execution. Default is `LogLevel.ERROR`.
  * `logger` - (deprecated)

Specification of the closure is defined in [SshSpec](src/main/groovy/org/hidetake/gradle/ssh/api/SshSpec.groovy).


Contributions
-------------

Thanks for contributions. Your issue reports or pull requests are welcome.
