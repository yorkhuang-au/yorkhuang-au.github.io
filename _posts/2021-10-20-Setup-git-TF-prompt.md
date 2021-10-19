## Setup Shell Prompt for Git and TF

In ~/.bashrc add
```
RESETCOLOR="$(echo -e "\001\033[0m\002")"
PURPLETEXT="$(echo -e "\001\033[35m\002")"
REDBGWHITETEXT="$(echo -e "\001\033[1;41;97m\002")"
GREENTEXT="$(echo -e "\001\033[1;32m\002")"
BOLDTEXT="$(echo -e "\001\033[1m\002")"

TFPURPLE="$(echo -e "$PURPLETEXT(TF:$RESETCOLOR")"
TFPURPLEEND="$(echo -e "$PURPLETEXT)$RESETCOLOR")"

function terraform_workspace_prompt()
{
    if [ -d .terraform ]; then
        terraform_workspace="$(command terraform workspace show 2>/dev/null)"

        if [[ $terraform_workspace == "prod"* ]] || \
            [[ $terraform_workspace == "PROD"* ]] || \
            [[ $terraform_workspace == "Prod"* ]]; then
            echo -e " $TFPURPLE$REDBGWHITETEXT${terraform_workspace}$RESETCOLOR$TFPURPLEEND"

        elif [[ $terraform_workspace == "dev"* ]] || \
            [[ $terraform_workspace == "DEV"* ]] || \
            [[ $terraform_workspace == "Dev"* ]]; then
            echo -e " $TFPURPLE$GREENTEXT${terraform_workspace}$RESETCOLOR$TFPURPLEEND"

        else
            echo -e " $TFPURPLE$BOLDTEXT${terraform_workspace}$RESETCOLOR$TFPURPLEEND"
        fi
    fi
}

declare -x PS1="\\[\\033]0;\$TITLEPREFIX:\$PWD\\007\\]\\n\\[\\033[32m\\]\\u@\\h \\[\\033[35m\\]\$MSYSTEM \\[\\033[33m\\]\\w\\[\\033[36m\\]\`__git_ps1\`\\[\\033[0m\\]\$(terraform_workspace_prompt)\\n\$ "
```
