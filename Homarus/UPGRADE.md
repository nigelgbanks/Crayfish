This document guides you through the process of upgrading Homarus. First, check if a section named "Upgrade to x.x.x" exists, with x.x.x being the version you are planning to upgrade to.

## Upgrade to 3.0.0

Homarus (and all of Crayfish) adheres to [semantic versioning](https://semver.org), which makes a distinction between "major", "minor", and "patch" versions. The upgrade path will be different depending on which previous version from which you are migrating.

### Upgrade from version 2.x.x

Homarus has switched from a Silex application to a Symfony application. This does not require much in code changes, but does use a different file layout.

Previously your configuration file would be located in the `/path/to/Homarus/config` directory and be called `config.yaml`.

The configuration from this file will now be located several locations documented below.

#### FFMpeg executable location
Old location `/path/to/Homarus/config/config.yaml`

```
---
homarus:
  # path to the ffmpeg executable
  executable: ffmpeg
```

The `executable` variable is now located in `/path/to/Homarus/config/services.yaml` and appears in the `parameters`

```
parameters:
    app.executable: ffmpeg
```

#### Mimetypes and mime\_to_format
Old location `/path/to/Homarus/config/config.yaml`

```
---
homarus:
  ...
  mime_types:
    valid:
      - video/mp4
      - video/x-msvideo
      - video/ogg
      - audio/x-wav
      - audio/mpeg
      - audio/aac
      - image/jpeg
      - image/png
    default: video/mp4
  mime_to_format:
    valid:
      - video/mp4_mp4
      - video/x-msvideo_avi
      - video/ogg_ogg
      - audio/x-wav_wav
      - audio/mpeg_mp3
      - audio/aac_m4a
      - image/jpeg_image2pipe
      - image/png_image2pipe
    default: mp4
```

The two lists (`mime_types.valid` and `mime_to_format.valid`) have been combined into a single list. The new variable is now located in `/path/to/Homarus/config/services.yaml` and appears in the `parameters`

The `mime_to_format` variable in version 2.x.x was the combination of the mimetype, the underscore character ( _ ) and the FFMpeg format (ie. `video/mp4_mp4`). In version 3.0.0 we create a list with keys `mimetype` and `format`.

```
parameters:
    ...
    app.formats.valid:
        - mimetype: video/mp4
          format: mp4
        - mimetype: video/x-msvideo
          format: avi
        - mimetype: video/ogg
          format: ogg
        - mimetype: audio/x-wav
          format: wav
        - mimetype: audio/mpeg
          format: mp3
        - mimetype: audio/aac
          format: m4a
        - mimetype: image/jpeg
          format: image2pipe
        - mimetype: image/png
          format: image2pipe
```

The two `default` variables (`mime_type.default`, `mime_to_format.default`) have been combined and moved to the `app.formats.defaults` variable

```
parameters:
    ...
    app.formats.defaults:
        mimetype: video/mp4
        format: mp4
```

#### Fedora Resource
Old location `/path/to/Homarus/config/config.yaml`

```
...
fedora_resource:
  base_url: http://localhost:8080/fcrepo/rest
```

This variable is necessary for the Crayfish-Commons setup, it has been moved to `/path/to/Homarus/config/packages/crayfish_commons.yaml`

```
crayfish_commons:
  fedora_base_uri: 'http://localhost:8080/fcrepo/rest'
```

#### Log settings
Old location `/path/to/Homarus/config/config.yaml`

```
...
log:
  # Valid log levels are:
  # DEBUG, INFO, NOTICE, WARNING, ERROR, CRITICAL, ALERT, EMERGENCY, NONE
  # log level none won't open logfile
  level: DEBUG
  file: /var/log/islandora/homarus.log
```

This setting is in `/path/to/Homarus/config/packages/monolog.yaml`. This file contains commented out defaults from Symfony and a new handler for Homarus.

```
monolog:
    handlers:
    ...
        homarus:
            type: rotating_file
            path: /tmp/Homarus.log
            level: DEBUG
            max_files: 1
            channels: ["!event", "!console"]
```

#### Syn settings
Old location `/path/to/Homarus/config/config.yaml`

```
syn:
  # toggles JWT security for service
  enable: True
  # Path to the syn config file for authentication.
  # example can be found here:
  # https://github.com/Islandora/Syn/blob/main/conf/syn-settings.example$
  config: ../syn-settings.xml
```

The `syn.enable` variable is no longer used as Syn is part of the security for Symfony, see [below](#enable-disable-syn) for steps to see where to enable/disable Syn.

The `syn.config` variable is in `/path/to/Homarus/config/crayfish_commons.yaml`.

```
crayfish_commons:
  ...
  #syn_config: '/path/to/syn-settings.xml'
```

`crayfish_commons.syn_config` needs to point to a file or be left commented out to use a default syn config of

```
<?xml version="1.0" encoding="UTF-8"?>
<!-- Default Config  -->
<config version='1'>
</config>
```

##### Enable/Disable Syn
To enable/disable Syn look in the [`./config/packages/security.yaml`](config/packages/security.yaml). By default Syn is disabled, to enable look the below lines and follow the included instructions

```
security:
    ...
    firewall:
        ...
        main:
            ...
            # To enable Syn, change anonymous to false and uncomment the lines further below
            anonymous: true
            ...
            # To enable Syn, uncomment the below 4 lines and change anonymous to false above.
            #provider: jwt_user_provider
            #guard:
            #    authenticators:
            #        - Islandora\Crayfish\Commons\Syn\JwtAuthenticator
```

