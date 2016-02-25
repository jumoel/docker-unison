# docker-unison

A Unison Docker container for fast file synchronization, i.e. on OSX where both
VirtualBox shares as well as NFS is awfully slow.

## Requirements

The only requirement is `unison`. It can be installed from your favorite
package manager such as `apt` or `brew`.

## Usage

With pure Docker (and, in this case, [dlite](https://github.com/nlf/dlite)):

```shell
$ UNISON=(docker run -d -p 5000:5000 jumoel/unison)
$ unison . socket://local.docker:5000/ -auto -batch
$ docker run --rm --volumes-from=$UNISON busybox ls -l /app
```

The `-auto` option for unison makes it run almost automatically and `-batch` makes
it run without waiting for user input. See `unison -help` for more options.

Usage with `docker-compose` is similarly simple:

```yaml
version: "2"
services:
  consumer:
    image: busybox
    command: sh -c "sleep 5 && ls -l /app"
    volumes_from:
      - unison
  unison:
    image: jumoel/unison
    ports:
      - "5000:5000"
```

To test it, run:

```shell
$ docker-compose up
```

In a new terminal, then hurry up and run

```shell
$ unison . socket://local.docker:5000/ -auto -batch
```

The `sleep 5 && ls -l /app` allows for time to synchronize the unison folder
before listing the contents of it. In a real scenario, the unison service will
probably need to be started separately, so an initial synchronization can be run
before the consumer service needs the files.

## Configuration

The container by default synchronizes to and shares the `/app` folder, but that
can be configured with the `UNISON_WORKING_DIR` [environment variable](https://docs.docker.com/engine/reference/run/#env-environment-variables).

Unison is set up as [`ENTRYPOINT`](https://docs.docker.com/engine/reference/builder/#entrypoint), so additional parameters when running the container
will be passed on to the program.
