octoprint
=========
[![Ansible Lint](https://github.com/oxivanisher/role-octoprint/actions/workflows/ansible-lint.yml/badge.svg)](https://github.com/oxivanisher/role-octoprint/actions/workflows/ansible-lint.yml)

Configures several things on octopi/octoprint hosts:
* Configure Webcam (old and new system)
* Copy timelapse files via rsync to a target server
* Disable webcam service by default. YOU HAVE TO MANUALLY add the following config to `/home/pi/.octoprint/config.yaml` or via the WebUi `Settings` > `Event Manager`:
  ```yaml
  events:
    enabled: true
    subscriptions:
    - command: sudo /bin/systemctl start camera-streamer.service
      event: PrintStarted
      type: system
    - command: sleep 30; sudo /bin/systemctl stop camera-streamer.service
      event: PrintDone
      type: system
    - command: sleep 30; sudo /bin/systemctl stop camera-streamer.service
      event: PrintFailed
      type: system
    - command: sleep 30; sudo /bin/systemctl stop camera-streamer.service
      event: PrintCancelled
      type: system
  ```
  This is for the new camera systemd, for the old one, the service is named differently. Since octoprint overrides the config file on stop, its complicated to do this via Ansible. Sorry, not really idempotent.

Role Variables
--------------

| Name                                      | Comment                                                                                           | Default value |
| ----------------------------------------- | ------------------------------------------------------------------------------------------------- | ------------- |
| `octoprint_timelapse_rsync_server`        | Rsync server hostname for timelapse sync                                                          |               |
| `octoprint_timelapse_rsync_user`          | Rsync user for timelapse sync                                                                     |               |
| `octoprint_timelapse_rsync_password`      | Rsync password for timelapse sync                                                                 |               |
| `octoprint_timelapse_rsync_path`          | Rsync module path for timelapse sync                                                              |               |
| `octoprint_timelapse_max_silent_failures` | Number of consecutive rsync failures to tolerate before the systemd service enters a failed state | `2`           |
| `octoprint_user`                          | OS user running OctoPrint (used to locate the timelapse directory)                                | `pi`          |

The timelapse sync runs as a systemd timer (`octoprint-timelapse-sync.timer`) every hour at minute 35 with a random delay of up to 150 seconds. Failures are tracked across runs in `/var/lib/octoprint-timelapse/failure_count` (`StateDirectory=`); the lockfile lives in `/run/octoprint-timelapse/` (`RuntimeDirectory=`). The unit only enters a failed state after `octoprint_timelapse_max_silent_failures` consecutive failures, which tolerates transient DNS or network outages.

Example Playbook
----------------

```yaml
- name: Octoprint
  hosts: octoprint_server
  collections:
    - oxivanisher.raspberry_pi
  roles:
    - role: oxivanisher.raspberry_pi.octoprint
```

License
-------

BSD

Author Information
------------------

This role is part of the [oxivanisher.raspberry_pi](https://galaxy.ansible.com/ui/repo/published/oxivanisher/raspberry_pi/) collection, and the source for that is located on [github](https://github.com/oxivanisher/collection-raspberry_pi).
