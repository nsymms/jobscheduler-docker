# ** Work in Progress **
# Docker setup of SOS Berlin's JobScheduler

I'm using stock docker images from SOS Berlin and the official PostgresSQL image.  All taken from DockerHub.

So here I've put together a docker-compose version of a simple setup that contains the database, JOC, Controller, and one agent.  I'd like to make a Kubernetes cluster / helm chart setup some day so that there could be agent and controller scaling, but that's a later project.

### Notes:
- SOS Berlin's docker images use alpine v3.15 at the time of this writing. And there is a [known bug](https://github.com/alpinelinux/docker-alpine/issues/156) with bash in alpine docker containers since v3.13 (maybe earlier?). Some of the `test` functions fail. It's weird. So I had to edit one of their bash scripts (which was a bourne shell script until recently) to get it to run.
- We use a fixed hostname in the `docker-compose.yml` for the `joc` service because its hostname is stored in the application (and referenced). The random docker-generated hostnames change every time the instance is created and would cause problems otherwise.
- **IMPORTANT**: The compose file uses docker volumes instead of simple local directories. This is used because there is pre-defined setup data inside the container images. When mounting a new empty volume, the container's contents at that mount point are copied into the new volume. The container then mounts and uses the volume. This initial data copy is **NOT** performed when using bind mounts (local directories & files).

### How to run this setup:
- Edit `config/hibernate.cfg.xml` and set db name/user/pass. Make sure it matches what's listed in `docker-compose.yml` for the postgresql service.
- Before first startup, the database must be initialized.  So do this:
```
docker-compose up db -d
docker-compose run --rm --entrypoint "/bin/sh -c /opt/sos-berlin.com/js7/joc/install/joc_install_tables.sh" joc
# Once complete, to start the full container set:
docker-compose up -d
```

- At first start, browse to the JOC at http://localhost:4446. The user / password is `root` / `root`.
- You will be prompted to attach the controller and agent. The controller URL is http://controller:4444. Once you've submitted that, the controller window will display and you'll need to add the agent. Click on the `...` and then "Add single agent". The agent url is http://agent:4445.
- After that is set up, you can navigate to the dashboard.

Further documentation for JobScheduler can be found at the SOS Berlin website here: https://kb.sos-berlin.com/display/PKB/JS7

