# beingtomgreen.plex_docker

A simple ansible role for running [Plex](https://www.plex.tv/) via Docker compose using the [LinuxServer.io Plex image](https://docs.linuxserver.io/images/docker-plex).

Since you're already here, why not look at [Jellyfin](https://jellyfin.org/) instead, it's like Plex, but better, I have a [role for that too](https://github.com/BeingTomGreen/ansible-role-jellfin-docker).

For additional information on how to best handle your media paths see information on [wiki.servarr.com](https://wiki.servarr.com/docker-guide#consistent-and-well-planned-paths) and [trash-guides.info](https://trash-guides.info/File-and-Folder-Structure/Hardlinks-and-Instant-Moves/).

## Installation

Given that Galaxy seems to have abandoned roles, I suggest referencing this repository directly in your projects `requirements.yml`:

```yaml
---

roles:
  - name: beingtomgreen.plex_docker
    src: https://github.com/BeingTomGreen/ansible-role-plex-docker.git

collections: []
```

You can then install it alongside your other requirements as normal:

```bash
ansible-galaxy install -r requirements.yml
```

## Example usage

### Basic usage

```yaml
---

- hosts: plex
  become: true
  vars:
    plex_docker_devices:
      - '/dev/dri:/dev/dri'
      - '/dev/dvb:/dev/dvb'

    plex_docker_env_puid: 5000
    plex_docker_env_pgid: 5000

    plex_docker_media_volumes:
        - '/path/to/tv:/tv'
        - '/path/to/movies:/movies'
        - '/path/to/music:/music'
  roles:
    - role: beingtomgreen.plex_docker
```
### Hardware Acceleration

Plex supports hardware acceleration with a GPU, but will require [additional config](https://docs.linuxserver.io/images/docker-plex/#hardware-acceleration):

#### Intel/ATI/AMD

If you're rocking an Intel, ATI or AMD card, setup is nice and simple, as you'd expect:

```yaml
---

- hosts: plex
  become: true
  vars:
    # Just pass through your device(s)
    plex_docker_devices:
      - '/dev/dri:/dev/dri'
  roles:
    - role: beingtomgreen.plex_docker
```

#### NVIDIA

If you're stuck with an NVIDIA GPU, you will need the [container toolkit](https://github.com/NVIDIA/nvidia-container-toolkit) installed, and then set the `runtime` to `nvidia`:

```yaml
---

- hosts: plex
  become: true
  vars:
    # Use the Nvidia container toolkit's runtime
    plex_docker_runtime: 'nvidia'

    # Optionally, set a comma separated list of device UUIDs or index(es)
    plex_docker_extra_environment_vars:
        NVIDIA_VISIBLE_DEVICES: '2,3'
  roles:
    - role: beingtomgreen.plex_docker
```

#### Arm Devices

In most cases if `/dev/dri` exists on the host, it should just work:

```yaml
---

- hosts: plex
  become: true
  vars:
    plex_docker_devices:
      - '/dev/dri:/dev/dri'
  roles:
    - role: beingtomgreen.plex_docker
```

If you're running a Raspberry Pi 4, be sure to enable `dtoverlay=vc4-fkms-v3d` in your `usercfg.txt`.

### Networking

LinuxServer.io recommends running their Plex image in a container with `network_mode: host`, so that's the default. But hey, who wants to be a sheep, you do you.

```yaml
---

- hosts: plex
  become: true
  vars:
    # Null network mode, since it can't be set if using networks, and is useless with ports
    plex_docker_network_mode:

    # Define our Docker ports, feel free to tweak as needed
    plex_docker_ports:
      - '32400:32400' # access to Plex Media Server - Required if not using network_mode: host
      - '1900:1900/udp' # access to the Plex DLNA Server
      - '5353:5353/udp' # older Bonjour/Avahi network discovery
      - '8324:8324' # controlling Plex for Roku via Plex Companion
      - '32410:32410/udp' # current GDM network discovery
      - '32412:32412/udp' # current GDM network discovery
      - '32413:32413/udp' # current GDM network discovery
      - '32414:32414/udp' # current GDM network discovery
      - '32469:32469' # access to the Plex DLNA Server

    # Alternatively, set a list of networks to attach this container to
    plex_docker_external_networks:
      - 'proxy'
      - 'my-little-network'
  roles:
    - role: beingtomgreen.plex_docker
```

### Want the kitchen sink?

Seriously, take a look at [`defaults/main.yml`](defaults/main.yml), it's obnoxiously commented, just for you.

```yaml
---

- hosts: plex
  become: true
  vars:
    plex_docker_cap_add:
      - 'CAP_WAKE_ALARM'
      - 'CAP_AUDIT_CONTROL'
    plex_docker_cap_drop:
      - 'CAP_CHECKPOINT_RESTORE'

    plex_docker_container_name: 'plex_container'

    plex_docker_depends_on:
      - 'my-little service'

    plex_docker_devices:
      - '/dev/dri:/dev/dri'
      - '/dev/dvb:/dev/dvb'

    plex_docker_env_puid: 5000
    plex_docker_env_pgid: 5000

    plex_docker_env_timezone: 'Europe/London'

    plex_docker_env_version: 'latest'

    plex_docker_env_plex_claim: 'my-secret-claim-code'

    plex_docker_extra_environment_vars:
      SOME_OTHER_VAR: 'A_VALUE'

    plex_docker_image_tag: '1.41.8'

    plex_docker_labels:
      traefik.enable: 'true'
      traefik.http.routers.traefik-https.entrypoints: 'https'

    plex_docker_network_mode: 'host'

    plex_docker_pull_policy: 'weekly'

    plex_docker_restart_policy: 'always'

    plex_docker_media_volumes:
      - '/path/to/tv:/tv'
      - '/path/to/movies:/movies'
      - '/path/to/music:/music'

    plex_docker_base_directory: '/opt/plex_docker'

    plex_docker_prune_images: true
    plex_docker_prune_until: '24h'

  roles:
    - role: beingtomgreen.plex_docker
```

## Role Variables

See [`defaults/main.yml`](defaults/main.yml) for more details.

## Requirements

- Docker & docker compose installed on the target host
- ansible_user will also need to be in the docker group

## Dependencies

- community.docker

## License

[MIT](LICENSE)

## Author Information

[Tom Green](https://github.com/BeingTomGreen)
