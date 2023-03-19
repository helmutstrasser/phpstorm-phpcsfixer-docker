# Get php-cs-fixer running within docker for PHPStorm

<!-- TOC -->
* [Quality Tools setup](#quality-tools-setup)
* [External Tools setup](#external-tools-setup)
* [PHPStorm bug with setting synchronize files after execution](#phpstorm-bug-with-setting-synchronize-files-after-execution)
<!-- TOC -->

To get full php-cs-fixer support working in PHPStorm, two independent parts have to be set up correctly.
One to highlight code which has to be fixed, and another to apply changes written to the file. And to get things working
within docker some particularities have to be considered.

## Quality Tools setup

Note! Settings within the PHP node are project specific. This does not apply to the External Tools settings, which
will be discussed below.

To set up the code validation is the easy part:

![](./images/screen001.png "Set up PHP CS Fixer within Docker")

Click the three dots left to "Show ignored files" and configure the 
Docker interpreter. A good description about this configuration can be found
<a href="https://ddev.readthedocs.io/en/latest/users/topics/phpstorm/">
on the DDEV documentation website</a>.

If you want to use your own ruleset (as seen in the screenshot above), notice to use the path within the container, not the local one.

## External Tools setup

As mentioned above, settings within the External Tools configuration apply to all projects, which is a problem and
[has been discussed for years](https://youtrack.jetbrains.com/issue/IDEA-120007/External-Tools-configuration-cant-be-saved-as-project-level-settings).
So we have to find an as generic as possible solution.

Create a new entry in Settings -> Tools -> External Tools:

![](./images/screen002.png "External tool setup")

... where "Program:" shows the path to a file we have to create in the root directory of the current project, with the following content:

```
#!/bin/bash
/usr/local/bin/docker exec container-name /usr/bin/php vendor/friendsofphp/php-cs-fixer/php-cs-fixer --config=.php-cs-fixer.php "$@"
```

Change paths according to your project. Use paths that can easily apply for any project. The custom
path is located within the external file (.php-cs-fixer-helper) and can be changed for the particular project.

Here comes the first banana skin. It looks like php-cs-fixer can not be run with docker compose.
Use docker exec, followed by the name of the docker container!<br>
Alternatively you can also use DDEV if applicable:

```
#!/bin/bash
ddev exec -s web php vendor/friendsofphp/php-cs-fixer/php-cs-fixer --config=.php-cs-fixer.php "$@"
```

... which benefits from not having to know the name of the container.

Save this file and enter the path to the file in the "Program:"-field 
shown above (... ./.php-cs-fixer-helper).

```
Don't forget to make this file executable!
```

In the "Arguments:"-field use a variable to tell PHPStorm to run php-cs-fixer just for the current file:
```
fix $RemoteProjectFileDir$
```

Do so, if you don't want php-cs-fixer to scan the complete project every time you
run the fixer.

The "Working directory" should read ``` $ProjectFileDir$ ```

Activate "Synchronize files after execution" (and read about a bug below).

That's it. You can create a shortcut to run php-cs-fixer. It's located
here: 

![](./images/screen003.png "Shortcut for tool")

## PHPStorm bug with setting "Synchronize files after execution"

Before PHPStorm version 2022.3.3 a bug prevented PHPStorm from reloading files changed from external tools
[Link to discussion and the bugfix](https://youtrack.jetbrains.com/issue/IDEA-309781/External-Tools-Synchronize-files-after-execution-doesnt-wait-for-the-command-to-finish-unless-the-console-is-open). If you can't or don't want to update, the following workaround will reload the file changed by an external tool. Just add
```
sleep 1
```
to the end of the helper file.
