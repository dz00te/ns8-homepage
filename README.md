# ns8-homepage

This is a [homepage](https://github.com/gethomepage/homepage) module for NethServer 8

## Install

Instantiate the module with:

    add-module ghcr.io/dz00te/homepage:1.0.0-alpha1

To get the latest (testing) version:


    add-module ghcr.io/dz00te/homepage:latest


The output of the command will return the instance name.
Output example:

    {"module_id": "homepage1", "image_name": "homepage", "image_url": "ghcr.io/nethserver/homepage:latest"}

## Configure

Let's assume that the homepage instance is named `homepage1`.

Launch `configure-module`, by setting the following parameters:
- `<MODULE_PARAM1_NAME>`: <MODULE_PARAM1_DESCRIPTION>
- `<MODULE_PARAM2_NAME>`: <MODULE_PARAM2_DESCRIPTION>
- ...

Example:

    api-cli run module/homepage1/configure-module --data '{}'

The above command will:
- start and configure the homepage instance
- (describe configuration process)
- ...

Send a test HTTP request to the homepage backend service:

    curl http://127.0.0.1/homepage/

## Smarthost setting discovery

Some configuration settings, like the smarthost setup, are not part of the
`configure-module` action input: they are discovered by looking at some
Redis keys.  To ensure the module is always up-to-date with the
centralized [smarthost
setup](https://nethserver.github.io/ns8-core/core/smarthost/) every time
homepage starts, the command `bin/discover-smarthost` runs and refreshes
the `state/smarthost.env` file with fresh values from Redis.

Furthermore if smarthost setup is changed when homepage is already
running, the event handler `events/smarthost-changed/10reload_services`
restarts the main module service.

See also the `systemd/user/homepage.service` file.

This setting discovery is just an example to understand how the module is
expected to work: it can be rewritten or discarded completely.

## Uninstall

To uninstall the instance:

    remove-module --no-preserve homepage1

## Testing

Test the module using the `test-module.sh` script:


    ./test-module.sh <NODE_ADDR> ghcr.io/nethserver/homepage:latest

The tests are made using [Robot Framework](https://robotframework.org/)

## UI translation

Translated with [Weblate](https://hosted.weblate.org/projects/ns8/).

To setup the translation process:

- add [GitHub Weblate app](https://docs.weblate.org/en/latest/admin/continuous.html#github-setup) to your repository
- add your repository to [hosted.weblate.org]((https://hosted.weblate.org) or ask a NethServer developer to add it to ns8 Weblate project
