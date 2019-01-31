![comploy](https://raw.githubusercontent.com/superDuperCyberTechno/comploy/master/header.png)

_comploy_ is a simple deployment script written in bash, made to co-operate with git. The premise is super simple; when set up and executed, _comploy_ will deploy your source code (up to last commit) to your designated server(s).

### Installation
To download a copy, simply run the following command in the root of your git project: 

```
wget https://raw.githubusercontent.com/superDuperCyberTechno/comploy/master/comploy && chmod +x comploy
```

Open the file and edit the `hosts` variable found in the config section. - Here you will also be able to assign ignored files that will **not** be synchronized with the server. Ignored files *must* be seperated by a space.

Before you can use _comploy_ effectively, you need to execute it on a repository with no pending commits (IE: no staged files). You will be warned and denied execution if this requirement is not met. This is necessary since _comploy_ will not deploy any uncommitted, changed files. This forces us to provide the server with a complete codebase from the beginning.


### Usage
You can pass a string argument (representing the commit message) to _comploy_ if you want to commit and deploy at the same time. That - of course - is dependent on files actually being staged prior to execution. _comploy_ will then commit (if string argument has been passed) and deploy all unchanged (committed) files to the server:

```
./comploy 'your commit message here'
```

Or, if you have already committed:

```
./comploy
```

An even simpler solution is to add the following alias to your `.bashrc`:

```
function cmp() {
    ./comploy "$1"
}
```

... enabling you to simply write `cmp` instead of `./comploy`.

### Notes
_comploy_ sets user/group as `www-data:www-data`, `755` rights for folders and  `644` for files on the entire project on synchronization.

_comploy_ needs the client machine to have SSH root access to the deployment server.
