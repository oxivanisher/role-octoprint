octoprint
=========

Configures several things on octopi/octoprint hosts:
* Configure Webcam (old and new system)
* Add a [oxiscript](https://github.com/oxivanisher/role-oxiscripts) backup job for octoprint settings
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

| Name          | Comment                              | Default value |
|---------------|--------------------------------------|---------------|
| timelapse_rsync_server | Rsync server for timelapse sync |          |
| timelapse_rsync_user  | Rsync user for timelapse sync |          |
| timelapse_rsync_password | Rsync password for timelapse sync |           |
| timelapse_rsync_path | Rsync path for timelapse sync |           |

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
