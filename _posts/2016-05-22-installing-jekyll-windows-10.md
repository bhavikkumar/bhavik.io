---
layout: post
title:  "Installing Jekyll 3 on Windows 10"
date:   2016-05-22 17:30:00 +1200
---
If you have found this page then I'm guessing you already know what Jekyll is, however the documentation for a Windows 10 installation of Jekyll v3 is a bit lacking and it is easier than I original thought it would be. 

If this doesn't work please feel free to contact me and I'll make the appropriate updates.

### Installation of Ruby
 - Install Ruby v2.3.0 using [RubyInstaller](http://rubyinstaller.org/downloads/)
    - During the installation process ensure that `Add Ruby executables to your PATH` is checked.

Once the installation is complete, open up command line or powershell and run the following commands.

 - Run `gem update`
 - Run `gem cleanup`
 
### Installation of Jekyll
It is very easy to install Jekyll and only requires the following command.

 - Run `gem install jekyll`

### Site Creation
Now you are ready to create and run your first Jekyll website. Running the following commands on the command prompt will create a new site and then serve the website.

 - `jekyll new my-site`
 - `jekyll serve`

You can now visit your Jekyll website on [http://localhost:4000](http://localhost:4000). You are now ready to use Jekyll.