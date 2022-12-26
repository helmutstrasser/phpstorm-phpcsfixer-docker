# Get php-cs-fixer running within docker for PHPStorm

In PHPStorm two independent parts have to be set up correctly to get full
php-cs-fixer support working. One to mark code which has to be fixed, and
another to apply changes written to the file. And to get things working
within docker some particularities have to be considered.

To set up the code validation is the easy part:

![](./images/screen001.png "Set up PHP CS Fixer within Docker")

Click the three dots left to "Show ignored files" and configure the 
Docker interpreter. There is a good description about this configuration
<a href="https://ddev.readthedocs.io/en/latest/users/topics/phpstorm/">
on the DDEV documentation website</a>.

For the second part create a new external tool in Settings -> Tools ->
External Tools:

![](./images/screen002.png "External tool setup")

... where "Program:" shows the path to a file with the following content:

```
#!/bin/bash
/usr/local/bin/docker exec container-name /usr/bin/php /var/www/html/vendor/bin/php-cs-fixer "$@"
```

Here comes the first banana skin. It looks like php-cs-fixer can not be run with docker compose.
Use docker exec, followed by the name of the docker container!
Save this file and enter the path to the file in the "Program:"-field 
shown above (... php-cs-fixer-helper). Don't forget to make this file
executable!

In the "Arguments:"-field mention the php-cs-fixer configuration file
(--config=...). The whole row for copy&paste reads:
```
--config=/var/www/html/.php-cs-fixer.php fix $RemoteProjectFileDir$
```

The working directory should read ``` $ProjectFileDir$ ```

That's it. You can create a shortcut to run php-cs-fixer. It's located
here: 

![](./images/screen003.png "Shortcut for tool")

## PHPStorm bug with setting "Synchronize files after execution"

Unfortunately the setting "Synchronize files after execution" is not
working as intended. So if you hit the shortcut to run the external tool,
nothing will happen, though php-cs-fixer did his job. But as we
let php-cs-fixer change the file within docker, PHPStorm doesn't 
recognize any changes, as long as we don't synchronize the current file.
As a workaround this command can be bound to a shortcut close to the
external tool we did before. Let's say we use option-shift-f for the 
php-cs-fixer and bind "Reload all from disk" to option-shift-g, the
changed file can be quickly synchronized after applying php-cs-fixer.