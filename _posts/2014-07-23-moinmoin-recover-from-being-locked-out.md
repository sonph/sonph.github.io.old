---
layout: post
title: MoinMoin: Recover from being locked out of a page #acl
category: posts
---
If you ever get locked out of a MoinMoin wiki page because you accidentally set the ACL (Access Control List) for it to be something along the line of

    #acl All: [...]

You will need to actually log in to the server hosting the engine and edit the page directly on the file system. This is because MoinMoin blocks all requests to the page, including those from wiki superusers.

Here's how to do it:

1. Log in to the server hosting the wiki engine.
2. Go to `[MoinMoin directory]/data/pages/[Your page]`, where `[MoinMoin directory]` is the directory that MoinMoin was installed into. If you don't know where it is, try `find`-ing or `locate`-ing the configuration file, `wikiconfig.py`. Also note that you may need special permission to do this step, depending on your server configuration.

        cd [MoinMoin directory]/data/pages/[Your page]

3. Edit the current revision of the page by finding out what the current revision is and edit it (you may replace vim with your editor of choice).

        sudo vim ./revisions/$(cat current)

4. Delete the first line containing the `#acl` (`dd` in `vim`), and save the file (`:x` in `vim`).
5. Reload your browser (and web browser if needed).
6. Profits! Problem fixed!

SP