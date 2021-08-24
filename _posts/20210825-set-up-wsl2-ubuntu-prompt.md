 ## Set up WSL2 Ubuntu Prompt
 
 In a git-bash, run
 
 ```
  declare -p PS1
 ```
 
 It shows something like:
 
 ```
 declare -x PS1="\\[\\033]0;\$TITLEPREFIX:\$PWD\\007\\]\\n\\[\\033[32m\\]\\u@\\h \\[\\033[35m\\]\$MSYSTEM \\[\\033[33m\\]\\w\\[\\033[36m\\]\`__git_ps1\`\\[\\033[0m\\]\\n\$ "
 ```

Add this to Ubuntu ~/.bashrc

Now, Ubuntu terminal will look like this:
```
york@XLW-5CD0259KJX  ~/myfolder1/myfolder2 (mygitbranch)
$
```
