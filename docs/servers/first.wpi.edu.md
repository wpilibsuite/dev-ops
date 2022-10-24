# first.wpi.edu

WPILib needs to host certain files on the web for public consumption. WPI generously donates web space for WPILib. We use this for hosting hosting content like the latest Javadoc and Doxygen documentation. This can also be used to host other evergreen files like videos for the main documentation site.

WPILib does not manage the webserver that actually serves the files. Any webserver changes must be requested through WPI ITS. Because WPI does not allow for SSH connections from outside of the WPI Campus Network, the server must automatically pull new content to it instead of another service pushing content to it. The current files that should be on the webserver can be found at https://github.com/wpilibsuite/wpilibsuite.github.io.

The actual web server runs on a computer named tholian.wpi.edu. This hostname can be useful for explaining where you need assistance from ITS.

# Manual Access

To access the files, WPI community members can login via SSH to the linux remote console. For information on how to do that, please see this ITS article https://hub.wpi.edu/article/256/connecting-to-the-linux-terminal-server-using-linux. Once authenticated, you can browse the files at `/www/docs/VirtualHosts/first.wpi.edu/`.

# Update Script

To continuously update the content of the web server, a task must be executed to periodicity check for file updates. This is very easy because the content is all stored in a git repository on GitHub.

An important note is that the WPI Linux terminal server (linux.wpi.edu) will not always connect you to the same server. As of writing there are 4 servers that are load balanced together. Instead of sshing to linux.wpi.edu, ssh into one of these servers and remember which one! The crontab file is not shared between them. Right now, the crontab is configured on `ccc-app-p-u04.wpi.edu`.

1. Before configuring the update script, you must complete an initial clone of the content manually. The files must have permissions of 766 in order for the web service to access them. To accomplish this we can set the `core.sharedRepository` configuration option to `umask` on initial clone.

```bash
git -c "core.sharedRepository=umask" clone https://github.com/wpilibsuite/wpilibsuite.github.io.git /www/docs/VirtualHosts/first.wpi.edu/wpilib
```

2. Now we can configure the server to check for updates every so often. We will use cron to accomplish this. We will set the update to happen every 15 minutes (0, 15, 30, 45 minutes of the hour). Open the crontab with:

```bash
crontab -e
```

and add the following entry

```
*/15 * * * * /usr/local/bin/git -C /www/docs/VirtualHosts/first.wpi.edu/wpilib pull -q
```

3. Save and close your editor to save the crontab.

4. You can check the content of the crontab by running

```bash
crontab -l
```

# A Note on Security

Sometimes WPI will use security settings that are not compatible with all IDEs. Every once in a while check the security settings of the server to make sure they have not changed. If they have, some of our clients will not connect to the server. The SSL Labs SSL Test tools is great at this.

https://ssllabs.com/ssltest/analyze.html?d=first.wpi.edu
