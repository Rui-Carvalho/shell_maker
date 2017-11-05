# Shell Maker

A powerful set of shell scripts that allows you to easily define commands to run for a project.

# Installation

1. Just checkout this repository to your local machine  
1. Copy the file `src/make_cmds.dist` to a file named `src/make_cmds`, ie, on same dir. This file will be your specific local project file, not versioned, with your own commands that you will define.
1. Run: `> chmod +x src/make`
1. That's it! Easy peasy

# What Is It?

* You create shell commands on the file make_cmds
* You give these commands a description and a set of shell commands to run.
* You run ./make and it will show you the list of available commands to run.
* Choose a number and it will run all the commands.
* You can show information in between run commands, you can ask confirmation from user, you can ask for variables.  


# What It Looks Like

![Menu Created by the Scripts](/menu.png?raw=true "Menu Created by the Scripts")


# Usage

* You can run from src folder `> ./make` or from bin folder: `> ./mk` as you prefer  
* You can run it in interactive mode: `> ./make -i` or simply `> ./make`  
* You can directly run a command you know the number of: `> ./make 1`  
* You can run commands 1, 2 and 3 in sequence: `> ./make 1 2 3`  

To make your life easier you can add an alias to your alias file - eg: ~/.bash_aliases - as in:  

```shell
alias mk='cd <path_to_this_repo_on_your_machine>/src && ./make'
```

# Defining The Commands For Your Project  

You define the commands you want to run in the file make_cmds  

Each command must have a number to it and follow the following structure for definition:  

* One line for the command description => will be used in the menu
* One line with __commands defined in between double quotes__

NOTE: if your command uses double quotes make sure you escape them - see below for example of Hello World.  

```shell
CMD_1_DESCRIPTION=""
CMD_1=( "<here you place the shell command to run>" )
```

You can use back slashes to define the commands in multiple lines, as in:

```shell
CMD_1_DESCRIPTION=""
CMD_1=( \
"echo \"Hello\"" \
"echo \"World\"" \
)
# ====================================================
```

Here is an example:

```shell
CMD_1_DESCRIPTION="Clear Symfony2 Cache (rm, cache:clear, chmod 777)"
CMD_1=( \
"sudo rm -Rf app/cache/*" \
"sf cache:clear" \
"sudo chmod -R 777 app/cache" \
)
```
You can use comments as you wish, for instance, you can use them to separate the commands definition as in:

```shell
# ====================================================
CMD_1_DESCRIPTION="Clear Symfony2 Cache (rm, cache:clear, chmod 777)"
CMD_1=( \
"sudo rm -Rf app/cache/*" \
"sf cache:clear" \
"sudo chmod -R 777 app/cache" \
)
# ====================================================
CMD_2_DESCRIPTION="Rebuild Site FE"
CMD_2=( \
"info-Rebuilds the Site FE app." \
"gulp dev")
# ====================================================
```

## NOTE: you can use the following
* `confirm[-<optional message to user>]`  ===> asks for confirmation at beginning of command sequence
* `continue` ==========> asks user to press any key to continue
* `info` ==============> informs user about any message you want to show
* `askIfSkip` =========> asks the user if he wants to skip the next command
* `forceRunAsk` =======> asks if user wants to run in force mode: runs all commands without confirmation
* `askVar-{COL}` ======> asks the user for a variable that can be used on further instructions


# DETAILED EXAMPLE WITH ALL OPTIONS BEING USED

```shell
#!/bin/bash

# ====================================================
CMD_1_DESCRIPTION="Say Hello World - Demonstrates usage of double quotes: must be escaped!"
CMD_1=( \
"echo \"Hello\"" \
"echo \"World\"" \
)
# ====================================================
CMD_2_DESCRIPTION="Rebuild Site FE - Demonstrates the use of backslash for multiline and not using it for inline"
CMD_2=( \
"info-Rebuilds the Site FE app." \
"gulp dev")
# ====================================================
CMD_3_DESCRIPTION="Update the FE (git, npm, bower, gulp, symlink, cache clear) - Demonstrates FE build scenario"
CMD_3=( \
"npm install" \
"bower install" \
"gulp dev" \
"sf assets:install --symlink" \
"sudo rm -Rf app/cache" \
"sf cache:clear" \
"sudo chmod -R 777 app/cache" \
)
# ====================================================
CMD_4_DESCRIPTION="Rebuild Users ES DB and import all"
CMD_4=( \
"sf elasticsearch:index:delete --index=users" \
"sf elasticsearch:profile:mapping:update --index=users --type=user --yolo" \
"sf user:queue:push --yolo --instant" \
)
# ====================================================
CMD_5_DESCRIPTION="Update All Schemas (all EMs) - Demonstrates using a function at bottom of this script"
CMD_5=( \
"update_all_schemas" \
)
# ====================================================
CMD_6_DESCRIPTION="Search for a column in all MySQL databases - Demonstrates asking for a variable and using it in a string passed to function"
SQL="SELECT TABLE_SCHEMA, TABLE_NAME, COLUMN_NAME, COLUMN_TYPE, COLUMN_COMMENT FROM INFORMATION_SCHEMA.COLUMNS WHERE COLUMN_NAME LIKE '%{COL}%'"
CMD_6=( \
"askVar-{COL}-What is the name of the column to find?" \
"exec_mysql_cmd \"$SQL\""
)
# ====================================================
CMD_7_DESCRIPTION="DB Creation - Demonstrates asking user to confirm a command AND asking for DB name"
CMD_7=( \
"confirm-Asks you for a new DB name to create on localhost" \
"read -r -p 'What is the name of the DB you want to create? ' option ; echo ${option} ; echo ';)'" \
)
# "mysql -u root -pmy_password -e 'CREATE DATABASE ${option}'" )
# ====================================================
CMD_8_DESCRIPTION="Show all services in Project - Demonstrates showing info to the user"
CMD_8=( \
"info-Get extra info on any Symfony service in the Project.\nScroll and get service number. Then use it to get info." \
"sf container:debug")
# ====================================================
CMD_9_DESCRIPTION="Fix Magento2 Permissions - (Prod or Staging) - Demonstrates asking for a variable called ENV"
CMD_9=( \
"askVar-{ENV}-Apply to: (1) Production or (2) Staging ?" \
"fixMagento2Permissions {ENV}" \
)
# ====================================================


# ====================================================
#             SUPPORTING FUNCTIONS NEEDED
# ====================================================
function exec_mysql_cmd {
    #echo "${1}"
    mysql -u root -p -h localhost -e "${1}"
}

function update_all_schemas {
    ems=(default users profiles newsletter)
    for em in ${ems[@]}
    do
        echo -e "\n ${yellow} >>> Updating schema for: $em ...${STD}"
        php app/console doctrine:schema:update --force --em=$em -e=devbox --no-debug;
	echo -e "\n"
    done
}
function fixMagento2Permissions {
	echo $1
}
```
