Hints for hacking on bestpractical.com's RT (Request Tracker)
======

GENERAL NOTES
-------------
Questions during hacking: Go read the code.
You may also try the documentation at https://docs.bestpractical.com/rt/4.4.1/index.html

There is some documentation, e.g. for version 4.4.1, here https://docs.bestpractical.com/rt/4.4.1/index.html
This gives an overview regarding available classes.

Then, locate the modules in question. You can do some print-style learning by printing in these scripts. STDOUT goes to the web page, STDERR to the server console.

LOCAL INSTALL
-------------

A local install with SQLite backend is easiest to set up.
Use the sdbs tool to build rt in ~/opt/rt4

    git clone https://github.com/oetiker/sdbs
    export PREFIX=~/opt/rt4
    sdbs/build_rt-4.4.1.sh

At the end of the installation, you should see:

    ...
    You must now configure RT by editing /home/your-home/rt4/etc/RT_SiteConfig.pm.
    ...

If you have any specific setting, include them.
You will certainly need DB access:

    Set( $rtname, 'picture test xyz');

    Set($DatabaseType, "SQLite");
    Set($DatabaseUser, "rt_user");
    Set($DatabasePassword, q{rt_pass});
    Set($DatabaseName, q{rt4});
    Set($DatabaseAdmin, "root");

Set the server name to your domain, or you will get a lot of cross-site warnings.

    Set($rtname, "your-domain.org");
    Set($Organization, "your-domain.org");

Set the web port to what you will run the debug server under.

    Set($WebPort, 12345);

If you want debug logging to file.

    Set($LogToFile , 'debug');
    Set($LogDir, 'log/');
    Set($LogToFileNamed , "rt.log");

Or, as an example, turn on SQL statment debugging and send log to STDERR

    Set($StatementLog, 'debug');
    Set($LogToSTDERR, 'debug');

Then, init the DB

    make initialize-database
    # if it complains that it could not write the log, check if log/ is created; if not
    mkdir ~/opt/rt4/log
    make initialize-database

During development, launch your local installation like so:

    cd ~/opt/rt4

    cd var/mason_data
    rm -rf obj/*
    PERL5LIB=/home/your-user/opt/rt4/lib:/home/your-user/opt/rt4/lib/perl5 /home/your-user/opt/rt4/sbin/rt-server --port 12345

This will clear the mason cache and start your rt in ~/opt/rt4 on port 12345.
Then, point your browser to localhost:12345

If you are working on a server and still want to see the local install in a browser on your laptop, ssh to the server like so:

    ssh -L 12345:localhost:12345 your-account@your-server.example.com

This will forward port 12345 to your laptop and localhost:12345 works in the browser.

Running the local server on a non-standard port will show a lot of cross-site issues.
Edit ~/opt/rt4/etc/RT_SiteConfig.pm

    Set($WebPort, 12345);

Logging to file:

    Set($LogToFile , 'debug');
    Set($LogDir, 'log/');
    Set($LogToFileNamed , "rt.log"); #log to rt.log

