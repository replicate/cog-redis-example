# Cog Redis worker example

_Docker Compose setup for running a Cog model with the built-in Redis worker, to emulate how the model is deployed on replicate.com, and to provide an end-to-end example of the Redis worker API._

You can use this project to debug the rare case where a model works locally but doesn't work on replicate.com, even though the development environment is the same as on replicate.com (11G RAM, T4 GPU).

## Usage

First, start the docker-compose processes, specifying the model image you want to test:

```
$ IMAGE=r8.im/replicate/hello-world@sha256:5c7d5dc6dd8bf75c1acaa8565735e7986bc5b66206b55cca93cb72c9bf15ccaa docker-compose up
```

In another shell, connect to Redis:

```
$ docker exec -it cog-redis redis-cli
```

Now you can queue up inputs in the format the model expects, e.g.

```
XADD input-queue * value '{"input": {"text": "world"}, "response_queue":"output-queue"}'
```

Shortly thereafter, outputs should be available on the output queue:

```
$ RPOP output-queue
"{\"status\": \"succeeded\", \"output\": \"hello world\", \"logs\": []}"
```

## Use during development

If you want to test the Redis worker during development, copy the file `docker-compose.yml` into your project's root directory.

Build the Cog model:

```
$ cog build
[...]
Successfully built cog-my-model
```

Save the image name to your clipboard (in this example it's `cog-my-model`).

Then you can start docker-compose:

```
$ IMAGE=cog-my-model docker-compose up
```

And now you can push messages to the queue as in the previous section.

## Use without GPU

By default the docker-compose file runs the model with GPU. If you don't have a GPU or don't need that, comment out the following lines from docker-compose.yml:

```
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: 1
              capabilities: [gpu]
```
