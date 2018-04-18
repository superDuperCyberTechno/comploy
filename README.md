![comploy](https://raw.githubusercontent.com/superDuperCyberTechno/comploy/master/header.png)

### *comploy* is a simple deployment script written in bash, made to co-operate with git.

The premise is super simple; when set up and executed, _comploy_ will deploy your source code (up to last commit) to your designated server.

You can pass a string argument (representing the commit message) to _comploy_ if you want to commit and deploy at the same time. That - of course - is dependent on files actually being staged prior to execution. _comploy_ will then commit (if string argument has been passed) and deploy all unchanged (committed) files to the server.

```
./comploy 'your commit message here'
```

Or, if you have already committed:

```
./comploy
```

#### Installation
To install, simply run the following command to download a copy: 

```
wget https://raw.githubusercontent.com/superDuperCyberTechno/comploy/master/comploy
```

Open the file and edit the _host_, - and potentially the _server_dir_ variables found in the config section. - Here you will also be able to assign ignored files that will **not** be synchronized with the server.

Before you can use _comploy_ effectively, you need to execute it on a repository with no pending commits (IE: no staged files). You will be warned and denied execution if this requirement is not met. This is necessary since comploy will not deploy any uncommitted, changed files. This forces us to provide the server with a complete codebase from the beginning.

In order to actually execute _comploy_ you might need to have execution rights on the file:


```
chmod +x comploy
```
