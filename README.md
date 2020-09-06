# Bose Soundtouch
[![hacs_badge](https://img.shields.io/badge/HACS-Custom-orange.svg)](https://github.com/custom-components/hacs)

Home Assistant custom component for Bose SoundTouch with snapshot and restore.

## Manual installation
1. Download the zip file and extract all files into your `config` directory.
2. Add the following section to your `configuration.yaml` and restart:

```yaml
# Example configuration.yaml
media_player:
  - platform: soundtouch
    host: 192.168.1.1
    port: 8090
    name: Soundtouch Living Room
```

## Hacs installation
1. Install hacs to your homeassistant installation. See https://hacs.xyz/docs/installation/manual
2. Add this repository to hacs: https://github.com/JoDehli/PyLoxone
3. Install the PyLoxone binding
4. Add the following section to your `configuration.yaml` and restart:

```yaml
# Example configuration.yaml
media_player:
  - platform: soundtouch
    host: 192.168.1.1
    port: 8090
    name: Soundtouch Living Room
```

## Features

All existing features of the original Component are supported:
https://www.home-assistant.io/integrations/soundtouch/

New - To make a snapshot from what us currently playing:
```yaml
service: soundtouch.snapshot
data:
  entity_id: media_player.living_room
```
(volume will by default also be added to the snapshot)

Then you can change the source and the volume.

New - To restore the earlier created snapshot:
```yaml
service: soundtouch.snapshot
data:
  entity_id: media_player.living_room
```
(volume will by default also be restored from the snapshot)

## Example
```yaml
script:
  # Script definition
  soundtouch_play_tts:
    alias: Soundtouch play TTS
    fields:
      entity:
        description: 'Entity Id of the media player to use'
        example: 'media_player.living_room'
      message:
        description: 'Message to speak.'
        example: 'This is a test message.'
      volume:
        description: 'Volume between 0 and 1'
        example: "0.1"
      delay:
        description: 'Time to take to play the message'
        example: '00:00:10'
    sequence:
      - service: soundtouch.snapshot
        data_template:
          entity_id: "{{entity}}"
      - service: media_player.volume_set
        data_template:
          entity_id: "{{entity}}"
          volume_level: "{{volume}}"
      - service: tts.google_translate_say
        data_template:
          entity_id: "{{entity}}"
          message: '{{message}}'
      - delay: "{{delay}}"
      - service: soundtouch.restore
        data_template:
          entity_id: "{{entity}}"

  # example to call script
  soundtouch_play_tts_example:
    alias: Soundtouch play TTS in Living Room
    sequence:
      - service: script.soundtouch_play_tts
        data:
          entity: media_player.living_room
          message: 'Someone is at the front door.'
          volume: 0.1
          delay: '00:00:10'
```