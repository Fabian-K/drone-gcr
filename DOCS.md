Use the Docker plugin to build and push Docker images to a registry.
The following parameters are used to configure this plugin:

* `registry` - authenticates to this registry (defaults to `gcr.io`)
* `token` - json key file
* `repo` - repository name for the image
* `tag` - repository tag for the image
* `storage_driver` - use `aufs`, `devicemapper`, `btrfs` or `overlay` driver

The following is a sample Docker configuration in your .drone.yml file:

```yaml
publish:
  docker:
    registry: gcr.io
    token: |
      $$GLOUD_KEY
    repo: foo/bar
    tag: latest
    file: Dockerfile
```

You may want to dynamically tag your image. Use the `$$BRANCH`, `$$COMMIT` and `$$BUILD_NUMBER` variables to tag your image with the branch, commit sha or build number:

```yaml
publish:
  gcr:
    registry: gcr.io
    token: |
      $$GLOUD_KEY
    repo: foo/bar
    tag: $$BRANCH
    file: Dockerfile
```

Or you may prefer to build an image with multiple tags:

```
publish:
  gcr:
    registry: gcr.io
    token: |
      $$GLOUD_KEY
    repo: foo/bar
    tag:
      - latest
      - "1.0.1"
      - "1.0"
```

Note that in the above example we quote the version numbers. If the yaml parser interprets the value as a number it will cause a parsing error.

## JSON Key

## Troubleshooting

For detailed output you can set the `DOCKER_LAUNCH_DEBUG` environment variable in your plugin configuration. This starts Docker with verbose logging enabled.

```
publish:
  gcr:
    environment:
      - DOCKER_LAUNCH_DEBUG=true
```

## Known Issues

There are known issues when attempting to run this plugin on CentOS, RedHat, and Linux installations that do not have a supported storage driver installed. You can check by running `docker info | grep 'Storage Driver:'` on your host machine. If the storage driver is not `aufs` or `overlay` you will need to re-configure your host machine.

This error occurs when trying to use the default `aufs` storage Driver but aufs is not installed:

```
level=fatal msg="Error starting daemon: error initializing graphdriver: driver not supported
```

This error occurs when trying to use the `overlay` storage Driver but overlay is not installed:

```
level=error msg="'overlay' not found as a supported filesystem on this host.
Please ensure kernel is new enough and has overlay support loaded." 
level=fatal msg="Error starting daemon: error initializing graphdriver: driver not supported"
```

This error occurs when using CentOS or RedHat which default to the `devicemapper` storage driver:

```
level=error msg="There are no more loopback devices available." 
level=fatal msg="Error starting daemon: error initializing graphdriver: loopback mounting failed" 
Cannot connect to the Docker daemon. Is 'docker -d' running on this host?
```

The above issue can be resolved by setting `storage_driver: vfs` in the `.drone.yml` file. This may work, but will have very poor performance as discussed [here](https://github.com/rancher/docker-from-scratch/issues/20).