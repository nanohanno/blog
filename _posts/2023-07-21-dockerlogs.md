---
layout: post
title: "Docker and linux processes"
subtitle: "Where are my logs, man?"
date: 2023-07-21
background: "images/default_post.jpg"
---
Docker logs seems so simple when it works. If logs are missing there are many things one needs to consider, especially in systems that on inherits. Hm, PYTHONUNBUFFERED? Logging handler? Debug level?

Wait a minute, could it be ´docker exec´? Yes, this time I learned a lot about linux processes and what docker exec actually does.

## Docker logs behind the scenes

Let's start with Docker. If a docker container is started with 

```bash
docker run --name python python:3.11 python -c "import logging; import time; logging.error('Start'); time.sleep(60); logging.error('End')"
```

the logging statements are handled by the default handler and sent to stdout and stderr. Those logs are picked up by the docker logging driver and shown on the terminal when executing `docker logs python`. Additionally, the logs are cached in the file system under `/var/lib/docker/containers/<container_id>` as json files per default. Those log files are then typically shipped to a central storage, for example with filebeat to elasticsearch.

## Adding some linux spice

In the container from the top, the waiting process is the main linux process. Every process has their own file descriptors for stderr and stdout. The main process will use `/proc/1/fd/1`. To play with this run

```bash
docker exec python /bin/sh -c "echo 'hello' >> /proc/1/fd/1"
```

and you see that in the running python container from the top a new log statement is shown (as long as you are a quick reader and the container is still alive otherwise restart the container). Any additional process will use a different file descriptor and logs will not show up on the stdout and stderr of the main process and consequently not in docker logs. Running a second process in the container will show that the logs are not ending up in the docker logs:

```bash
docker exec python python -c "import logging; logging.error('Second process')"
```

Those logs vanish in the container nirvana ending up in a file descriptor that will never be seen.

<iframe src="https://giphy.com/embed/l0HehB8QWj2iRwY8g" width="480" height="256" frameBorder="0" class="giphy-embed" allowFullScreen></iframe><p><a href="https://giphy.com/gifs/l0HehB8QWj2iRwY8g">via GIPHY</a></p>

## Recap

Docker has so many levels of understanding to it that you can always go a level deeper to understand things that were abstracted away and you never needed to care for it. 

A lot of irritation, a lot to learn :book:


## Resources

https://docs.docker.com/config/containers/logging/
https://www.baeldung.com/ops/docker-logs
