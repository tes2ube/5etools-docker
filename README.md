This is a simple image for hosting your own 5eTools instance. It is based on the Apache `httpd` image and uses a heavily-modified version of the auto-updater script from the [5eTools wiki](https://wiki.5e.tools/index.php/5eTools_Install_Guide).

# Usage with Docker Run

## Quickstart
You can quick-start this image by running:
`docker run -d -p 80:80 --rm --name 5etools-docker -v 5etools:/usr/local/apache2/htdocs jafner/5etools-docker`
Then give the container a minute or two to come online and it will be accessible at `localhost`.
When you stop the container, it will automatically delete itself, but the downloaded files will remain in the 5etools volume, so you can always start the container back up by re-running the command.

### Getting token images
You can configure the container's initialization script to download image files by setting the `IMG` environment variable:
`docker run -d -p 80:80 --rm --name 5etools-docker -v 5etools:/usr/local/apache2/htdocs -e IMG=true jafner/5etools-docker`
This will add a significant amount of time to the container's initialization.
The `-e` flag specifies a Docker environment variable, which is passed into the container's shell environment and allows for customization of the container during the run command. Here, the environment variable is named `IMG` and this command sets the value to `true`.

### Using a persistent volume
By default, this container uses a Docker-managed persistent volume for the server files. This saves allows the downloaded 5eTools files to persist, even if the container is destroyed. Alternatively, you can use a host directory mapping to share files between you host filesystem and the container. 

#### Using a host directory mapping 
You can alternatively persist your container's data by mapping a directory in the host's file system into the container. Assuming you want to use the directory `~/5etools-docker` on the host:

1. Create the directories with `mkdir ~/5etools-docker ~/5etools-docker/htdocs ~/5etools-docker/htdocs/download`. All of these directories need to be created for the mapping to work. 
2. Run the container with `docker run -d -p 80:80 --rm --name 5etools-docker -v ~/5etools-docker/htdocs:/usr/local/apache2/htdocs`
Note: host directory mappings must be absolute (cannot use `.` to refer to working directory). However, you can still refer to your working directory with `${PWD}`. 

### Updating the container
Because this image is built on the auto-updater script, updating the container is very simple. Restart the container with `docker restart 5etools-docker`. When it restarts, the container will automatically check for an update and download it before starting. 
Note: there is no way to disable this auto-updating behavior except to never restart the container. If you want a specific version of the container, it is recommended that you look into using the `httpd` image instead.

### Integrating a reverse proxy
Supporting integration of a reverse proxy is beyond the scope of this guide. 
However, any instructions which work for the base `httpd` (Apache) image, should also work for this, as it is minimally different.

# Usage with Docker Compose
Create the `~/5etools-docker/docker-compose.yml` file with your preferred text editor. Then add the following contents:

```yml
version: '3'
services:
  5etools:
  	image: jafner/5etools-docker
  	container_name: 5etools-docker
  	volumes:
      # to switch from a Docker-managed volume to a host directory mapping,
      # comment out the "- 5etools:..." line, 
      # and uncomment the "- ~/5etools-docker/htdocs:..." line
      # make sure the ~/5etools-docker/htdocs/download folder exists before onlining the stack
  	  - 5etools:/usr/local/apache2/htdocs # comment this line
      #- ~/5etools-docker/htdocs:/usr/local/apache2/htdocs # uncomment this line
  	ports:
  	  - 80:80
volumes:
  5etools:
```

This version has a persistent Docker-managed volume. If you would like to auto-load homebrew, you will need to switch to a host directory mapping, following the instructions in the Compose file.

# Auto-loading homebrew
To use auto-loading homebrew, you will need to use a host directory mapping as described above. 

1. Start the container (using either `docker run` or `docker-compose`) and wait for the container to finish starting. You can monitor its progress with `docker logs -f 5etools-docker`.
2. Assuming you are using the mapping `~/5etools-docker/htdocs:/usr/local/apache2/htdocs` place your homebrew json files into the `~/5etools-docker/htdocs/homebrew/` folder, then add their filenames to the `~/5etools-docker/htdocs/homebrew/index.json` file.
For example, if your homebrew folder contains:
```
index.json
'Jafner; JafnerBrew Campaigns.json'
'Jafner; JafnerBrew Collection.json'
'Jafner; Legendary Tomes of Knowledge.json'
'KibblesTasty; Artificer (Revised).json'
```
Then your `index.json` should look like:
```json
{
    "readme": [
        "NOTE: This feature is designed for use in user-hosted copies of the site, and not for integrating \"official\" 5etools content.",
        "The \"production\" version of the site (i.e., not the development ZIP) has this feature disabled. You can re-enable it by replacing `IS_DEPLOYED = \"X.Y.Z\";` in the file `js/utils.js`, with `IS_DEPLOYED = undefined;`",
        "This file contains as an index for other homebrew files, which should be placed in the same directory.",
        "For example, add \"My Homebrew.json\" to the \"toImport\" array below, and have a valid JSON homebrew file in this (\"homebrew/\") directory."
    ],
    "toImport": [
        "Jafner; JafnerBrew Collection.json",
        "Jafner; JafnerBrew Campaigns.json",
        "Jafner; Legendary Tomes of Knowledge.json",
        "KibblesTasty; Artificer (Revised).json"
    ]
}
```
Note the commas after each entry except the last in each array.
See the [wiki page](https://wiki.5e.tools/index.php/5eTools_Install_Guide) for more information. 