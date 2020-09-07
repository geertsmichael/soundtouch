# Bose Soundtouch
[![hacs_badge](https://img.shields.io/badge/HACS-Custom-orange.svg)](https://github.com/custom-components/hacs)

Home Assistant custom component for Bose SoundTouch with snapshot, restore and handoff.

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
2. Add this repository to hacs: https://github.com/geertsmichael/soundtouch
3. Install the Soundtouch integration
4. Add the following section to your `configuration.yaml` and restart:

```yaml
# Example configuration.yaml
media_player:
  - platform: soundtouch
    host: 192.168.1.1
    port: 8090
    name: Soundtouch Living Room
```

## Features Original

All existing features of the original Component are supported:
https://www.home-assistant.io/integrations/soundtouch/

## Features Snapshot & Restore

1. To make a snapshot from what us currently playing:
```yaml
service: soundtouch.snapshot
data:
  entity_id: media_player.living_room
```
(volume will by default also be added to the snapshot)

2. Then you can change the source and the volume.

3. To restore the earlier created snapshot:
```yaml
service: soundtouch.restore
data:
  entity_id: media_player.living_room
  restore_volume: false
```
(volume will by default also be restored from the snapshot, unless 'restore_volume' is set to false)

## Feature Handoff

The same snapshot and restore services can be used for an handoff.
An handoff is transitioning what is playing on one device to another device. 
And after the transition, turn off the original device.

```yaml
service: soundtouch.handoff
data:
  from: media_player.living_room
  to: media_player.bathroom
```

If you only want to transition the snapshot to another device, but not restore:

```yaml
service: soundtouch.handoff
data:
  from: media_player.living_room
  to: media_player.bathroom
  snapshot_only: true   
```
You need to restore the snapshot yourself on the 'to' device:
```yaml
service: soundtouch.restore
data:
  entity_id: media_player.bathroom
  restore_volume: false
```
(and if desired, turn it off, or first do something else on the device before turning it off)
(e.g. play a beep or text-to-speech message, see example)


## Example Snapshot & Restore
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
        example: '0.1'
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
          message: "{{message}}"
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

## Example Handoff
```yaml
script:
  # Script definition
  soundtouch_handoff_with_tts:
    alias: Soundtouch Handoff with TTS
    fields:
      entity_from:
        description: 'Entity Id of the media player to snapshot'
        example: 'media_player.living_room'
      entity_to:
          description: 'Entity Id of the media player to restore the snapshot on'
          example: 'media_player.bathroom'
      message:
        description: 'Message to speak.'
        example: 'This is a test message.'
      volume:
        description: 'Volume between 0 and 1'
        example: '0.1'
      delay:
        description: 'Time to take to play the message'
        example: '00:00:10'
    sequence:
      - service: soundtouch.handoff
        data_template:
          from: "{{entity_from}}"
          to: "{{entity_to}}"
          snapshot_only: true
      - service: soundtouch.restore
        data_template:
          entity_id: "{{entity_to}}"
          restore_volume: false
      - service: media_player.volume_set
        data_template:
          entity_id: "{{entity_from}}"
          volume_level: "{{volume}}"
      - service: tts.google_translate_say
        data_template:
          entity_id: "{{entity_from}}"
          message: "{{message}}"
      - delay: "{{delay}}"
      - service: media_player.turn_off
        data_template:
          entity_id: "{{entity_from}}"

  # example to call script
  soundtouch_handoff_with_tts_example:
    alias: Soundtouch handoff from Living Room to Bathroom with TTS
    sequence:
      - service: script.soundtouch_handoff_with_tts
        data:
          entity_from: media_player.living_room
          entity_to: media_player.bathroom
          message: 'Handoff to the bathroom completed. Enjoy!'
          volume: 0.1
          delay: '00:00:10'
```