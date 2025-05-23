```
 ██████╗ ██████╗ ███╗   ███╗██████╗ ██╗      ██████╗ ██╗   ██╗
██╔════╝██╔═══██╗████╗ ████║██╔══██╗██║     ██╔═══██╗╚██╗ ██╔╝
██║     ██║   ██║██╔████╔██║██████╔╝██║     ██║   ██║ ╚████╔╝
██║     ██║   ██║██║╚██╔╝██║██╔═══╝ ██║     ██║   ██║  ╚██╔╝
╚██████╗╚██████╔╝██║ ╚═╝ ██║██║     ███████╗╚██████╔╝   ██║
 ╚═════╝ ╚═════╝ ╚═╝     ╚═╝╚═╝     ╚══════╝ ╚═════╝    ╚═╝
./comploy 'commit/deploy' 
```
_comploy_ is a Linux-only, simple, web development deployment script written in Bash, made to co-operate with Git. The premise is super simple; when set up and executed, _comploy_ will deploy your source code (up to last commit) to your designated server(s).

It's kinda like an ultra lean [Deployer](https://deployer.org/)...

It plays nice with [Composer](https://getcomposer.org/) and [Laravel](https://laravel.com/). It's mostly geared towards PHP projects.

### Installation
_comploy_ works on a per-project basis so you need a copy for each of your projects you need to deploy.
To install it in your project, simply run the following command in the root of your git project:

```
wget https://raw.githubusercontent.com/superDuperCyberTechno/comploy/master/comploy && chmod +x comploy
```

Open the file and edit the `hosts` variable found in the config section. - Here you will also be able to assign ignored folders/files that will **not** be synchronized with the server. Ignored folders/files *must* be seperated by a space.

_comploy_ will use the default machine key (\~/.ssh/id_rsa) to deploy. Optionally you can define the absolute local path to another SSH key (the `key` variable) for this purpose.

Before you can use _comploy_ effectively, you need to execute it on a repository with no pending commits (IE: no staged files). You will be warned and denied execution if this requirement is not met. This is necessary since _comploy_ will not deploy any uncommitted, changed files. This forces us to provide the server with a complete codebase from the beginning.

### Uninstallation
_comploy_ is just one file. Just delete _comploy_ from your project and it's gone.

### Usage
Simply running _comploy_ will sync your project with the server, up to - but not including - files that have been added or changed since last commit. If you have a clean project (all files committed), it will sync everything.

You can also bypass the Git integration by setting `use_git` to `false`. In that case, everything (except ignored of course) is synced, every time. If `use_git` is true, _comploy_ will also try to push the codebase to your remote repository.

You can pass a string argument (representing the commit message) to _comploy_ if you want to commit, push __and__ deploy at the same time. That - of course - is dependent on files actually being staged prior to execution. _comploy_ will then commit (if string argument has been passed) and deploy all unchanged (committed) files to the server:

```
./comploy 'your commit message here'
```

Or, if you have already committed manually:

```
./comploy
```

An even simpler solution is to add the following function to your `.bashrc`:

```
cmp() {
    ./comploy "$1"
}
```

... enabling you to simply write `cmp` instead of `./comploy`.

### Requirements
* Client side, [_rsync_](https://rsync.samba.org/) is required, Git and Composer is optional.

### Notes
* _comploy_ sets user/group as `www-data:www-data`, `755` rights for folders and  `644` for files on the entire project folder on the server, when deploying.
* _comploy_ needs the client machine SSH key to have root access to the deployment server (to streamline the permission setup).
* _comploy_ will install the production version of the application (when using _composer_), locally. Then it syncs the project files and finally, re-installs the development version locally. This ensures local/server parity.
* You need to know how to set up a webserver in order to use _comploy_ effectively.
