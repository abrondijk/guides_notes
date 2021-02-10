# Plex tips and tricks

Honestly just [this](https://forums.plex.tv/t/linux-tips/276247) whole website. Lots of useful tips.

[This](https://gist.github.com/pjobson/3811b73740a3a09597511c18be845a6c) as well. Particularly the cronjob to edit the library file permissions.

In case the comment gets deleted, this is the contents of the shell script:

```bash
#!/bin/bash

# chmod files
# finds files which are not 664 permission and fixes them
find /mediaLibrary/movies        -type f \! -perm 664 -exec chmod 664 {} \; -print
find /mediaLibrary/shows         -type f \! -perm 664 -exec chmod 664 {} \; -print
find /mediaLibrary/music         -type f \! -perm 664 -exec chmod 664 {} \; -print

# chmod directories
# finds directories which are not 777 and fixes them
find /mediaLibrary/movies        -type d \! -perm 775 -exec chmod 775 {} \; -print
find /mediaLibrary/shows         -type d \! -perm 775 -exec chmod 775 {} \; -print
find /mediaLibrary/music         -type d \! -perm 775 -exec chmod 775 {} \; -print

# chown everything
# finds anything not owned by plex and fixes them
find /mediaLibrary/movies        \! -user plex -exec chown plex.plex {} \; -print
find /mediaLibrary/shows            \! -user plex -exec chown plex.plex {} \; -print
find /mediaLibrary/music         \! -user plex -exec chown plex.plex {} \; -print
```

Adding the following to the crontab will make it run every 30 minutes. This is efficient because it doesnt just update all the files, it only updates the files that dont thave those permissions yet.
30 seconds might be a bit overkill, adjust to your liking.

`*/30 * * * * /root/bin/plexOwn.sh >/dev/null 2>&1`
