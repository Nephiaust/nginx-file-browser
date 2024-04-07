# nginx file browser

This web application is a very simple file browser which can be used
effectively together with [nginx's autoindex module](http://nginx.org/en/docs/http/ngx_http_autoindex_module.html).

![nginx file browser in action - light theme](assets/screenshot-light.jpg)
![nginx file browser in action - light theme](assets/screenshot-dark.jpg)

A sample nginx configuration is also included which mounts **file browser** under root (`/`) and mounts files to be listed under `/files` path. Hence is the `filesBaseUrl` under

## Installation / Configuration

The system has a default to use the location's root (E.G. the site's root location `www.example.net/`, and the physical of `/opt/www/`). This can be configured to move both the ***installation*** location and the location to be browseable.

1. Clone the repo to the server
2. Create an installation location (E.G. `/var/www/files/`)
3. Copy the `css`, `image`, `js` directories to the installation location
4. Copy the `index.html` to the installation location
5. Create the location for the files to be shared & browsable (E.G. `/var/www/isos`)
6. Copy the files to the location
7. Make sure the files in the installation location & the location for the files to be shared are owned by the nginx process (E.G. `www-data:www-data` for Debian/Ubuntu based saystems)
8. Edit the nginx configuration file for the site (E.G. `/etc/nginx/sites-enabled/default`)
9. Create a location called files (see the example configuration below) and set the options appropriately
```
location /files {
    root /var/www/files/;
    index index.html index.htm;
}
```
10. Create a location for the files listing to be read by the application
```
location /myfiles {
    autoindex on;
    autoindex_format json;
}
```
11. Save the configuration and reload (or restart) the nginx process (E.G. `serivce nginx reload`)
12. Before loading the page to verify its working the `main.js` javascript file needs to be updated.
13. Edit the `main.js` file from the `<INSTALLDIR>/js/`
14. Find the line containing `var filesBaseUrl = "/files";` (about line 217). Change the `/files` to match the location from step 10.
15. Save the file
16. Loading the site now (E.G. `www.example.net/files`) will now load and show (as seen in the example pictures above)

## Using with docker

Mainly for demonstration purposes a docker image is also available [here](https://hub.docker.com/r/mohamnag/nginx-file-browser/).
In order to use this docker image, the volume which has to be served should
be mounted under `/opt/www/files/` and port `80` (root) or `8080` (rootless)) of container shall be mapped
to a proper port on host. A proper run would look like:

root
```
$ docker run -p 8080:80 -v /path/to/my/files/:/opt/www/files/ mohamnag/nginx-file-browser
```
rootless:
```
$ docker run -p 8080:8080 -v /path/to/my/files/:/opt/www/files/ mohamnag/nginx-file-browser
```

With container up and running you can point your browser to IP of docker host with given port to view the files. For example with above run command assuming docker host having IP with `192.168.0.200` we have to navigate to this URL:

`http://192.168.0.200:8080`


## Symlinks

> Be very careful with symlinks, they can expose very important files of system to outside world!

If you have symlinks inside files dir that you want to be able to browse too, the alias path where `/files` is served by nginx has to be changed to match the same path outside your docker container. Lets say I have a directory with path `/home/myuser/files-to-serve/`. Which has two directories named `dir1` and `dir2`. where `dir1` is nothing more than a symlink to `dir2`. In order to be able to browse `dir1` (inside `dir2`) on file browser, following have to be done:

Inside `default.conf` this line
```
    alias /opt/www/files/;
```

shall be changed to
```
    alias /home/myuser/files-to-serve/;
```

And the mounting point is now `/home/myuser/files-to-serve/` instead of `/opt/www/files/`.
