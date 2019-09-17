# Lovelace Xiaomi Vacuum Map card

[![hacs_badge](https://img.shields.io/badge/HACS-Default-orange.svg)](https://github.com/custom-components/hacs)
[![Community Forum](https://img.shields.io/badge/community-forum-brightgreen.svg?style=popout)](https://community.home-assistant.io/t/xiaomi-vacuum-interactive-map-card/123901)

This card enables you to specify target or start zoned cleanup using map, just like in Mi Home app. Additionally you can define a list of zones and choose ones to be cleaned.

## Go to target
![Go to target](https://github.com/PiotrMachowski/Home-Assistant-Lovelace-Xiaomi-Vacuum-Map-card/blob/master/s1.gif)

## Zoned cleanup
![Zoned cleanup](https://github.com/PiotrMachowski/Home-Assistant-Lovelace-Xiaomi-Vacuum-Map-card/blob/master/s2.gif)

## Defined zones
![Defined zones](https://github.com/PiotrMachowski/Home-Assistant-Lovelace-Xiaomi-Vacuum-Map-card/blob/master/s3.gif)

## Configuration options

| Key | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `entity` | `string` | `True` | - | ID of Xiaomi vacuum entity |
| `map_image` | `string` | `True` | - | Path to image of map |
| `calibration_points` | `List` | `True` | - | Pairs of coordinates: in vacuum system and on map image. See: [Calibration](#calibration)  |
| `zones` | `List` | `False` | Empty | List of predefined zones |
| `modes` | `List` | `False` | `[go_to_target, zoned_cleanup, predefined_zones]` | List of displayed modes. Possible values: `go_to_target`, `zoned_cleanup`, `predefined_zones` |
| `default_mode` | `string` | `False` | - | Default selected mode. Possible values: `go_to_target`, `zoned_cleanup`, `predefined_zones` |
| `debug` | `boolean` | `False` | `false` | Enables alerts with coordinates after holding `Start` button. Possible values: `true`, `false` |
| `service` | `string` | `False` | `vacuum.send_command` | Allows to define service used after clicking `Start` button. See: [Defining service](#defining-service) |
| `ignore_zones_limit` | `boolean` | `False` | `false` | Disables 5 zones limit  |

## Example usage:
```yaml
views:
- name: Example
  cards:
    - type: custom:xiaomi-vacuum-map-card
      entity: vacuum.xiaomi_vacuum
      map_image: '/local/custom_lovelace/xiaomi_vacuum_map_card/map.png'
      calibration_points:
        - vacuum:
            x: 25500
            y: 25500
          map:
            x: 466
            y: 1889
        - vacuum:
            x: 26500
            y: 26500
          map:
            x: 730
            y: 1625
        - vacuum:
            x: 25500
            y: 26500
          map:
            x: 466
            y: 1625
      zones:
        - [[25500, 25500, 26500, 26500]]
        - [[24215, 28125, 29465, 32175]]
        - [[24245, 25190, 27495, 27940], [27492, 26789, 28942, 27889]]
        - [[28972, 26715, 31072, 27915], [29457, 27903, 31107, 29203], [30198, 29215, 31498, 31215], [29461, 31228, 31511, 32478]]
```

## Installation
1. Download [*xiaomi-vacuum-map-card.js*](https://github.com/PiotrMachowski/Home-Assistant-Lovelace-Xiaomi-Vacuum-Map-card/raw/master/dist/xiaomi-vacuum-map-card.js), [*coordinates-converter.js*](https://github.com/PiotrMachowski/Home-Assistant-Lovelace-Xiaomi-Vacuum-Map-card/raw/master/dist/coordinates-converter.js), [*texts.js*](https://github.com/PiotrMachowski/Home-Assistant-Lovelace-Xiaomi-Vacuum-Map-card/raw/master/dist/texts.js) and [style.js](Home-Assistant-Lovelace-Xiaomi-Vacuum-Map-card/raw/master/dist/style.js) to `/www/custom_lovelace/xiaomi_vacuum_map_card` directory:
    ```bash
    mkdir -p www/custom_lovelace/xiaomi_vacuum_map_card
    cd www/custom_lovelace/xiaomi_vacuum_map_card/
    wget https://github.com/PiotrMachowski/Home-Assistant-Lovelace-Xiaomi-Vacuum-Map-card/raw/master/dist/xiaomi-vacuum-map-card.js
    wget https://github.com/PiotrMachowski/Home-Assistant-Lovelace-Xiaomi-Vacuum-Map-card/raw/master/dist/coordinates-converter.js
    wget https://github.com/PiotrMachowski/Home-Assistant-Lovelace-Xiaomi-Vacuum-Map-card/raw/master/dist/texts.js
    wget https://github.com/PiotrMachowski/Home-Assistant-Lovelace-Xiaomi-Vacuum-Map-card/raw/master/dist/style.js
    ```
2. Add card to resources in `ui-lovelace.yaml` or in raw editor if you are using frontend UI editor:
    ```yaml
    resources:
      - url: /local/custom_lovelace/xiaomi_vacuum_map_card/xiaomi-vacuum-map-card.js
        type: module
    ```

## Calibration

To calibrate this card you have to provide exactly 3 pairs of coordinates.

Each pair must consist of:
* coordinates of point in vacuum system (extracted by FloleVac or just by sending vacuum to a desired point)
* coordinates of matching point on a map image

For the best outcome calibration points should form a right triangle.

If you have used this card before a migration guide will appear instead of actual card.
## Defining service

You can use `service` parameter for example to run a script instead of starting a vacuum directly. Provided service will be run with following parameters:
* `entity_id` - id of a vacuum
* `command` - one of two:
  * `app_goto_target` - for _Go to target_ mode
  * `zoned_cleanup` - for _Zoned cleanup_ and _Predefined zones_ modes
* `params` - point or a list of zones (the same value as displayed in `debug` mode)

| WARNING: in current version of Home Assistant service `vacuum.send_command` does not support templates! |
| --- |
| To overcome this issue you can use a [*python script*](https://github.com/PiotrMachowski/Home-Assistant-Lovelace-Xiaomi-Vacuum-Map-card/raw/master/examples/vacuum_send_command.py). |

Example HA script that can be used with this card is available [*here*](https://github.com/PiotrMachowski/Home-Assistant-Lovelace-Xiaomi-Vacuum-Map-card/raw/master/examples/vacuum_send_command.yaml).

## Hints
* To find out values for `calibration_points` you can use service `vacuum.send_command` with data:
  ```json
  {
    "entity_id": "vacuum.xiaomi_vacuum",
    "command": "app_goto_target",
    "params": [25500, 25500]
  }
  ```
  Alternatively you can use `vacuum.xiaomi_clean_zone`:
    ```json
    {
      "entity_id": "vacuum.xiaomi_vacuum",
      "zone": [[25500, 25500, 26500, 26500]],
      "repeats": 1
    }
    ```
* You can find out coordinates for zones using two methods:
  * Enabling `debug` in settings, drawing zone in `Zoned cleanup` mode and holding `Start` button. Note: this method also works for other modes.
  * Android App [*FloleVac*](https://play.google.com/store/apps/details?id=de.flole.xiaomi)

* For Polish version download [*textsPL.js*](https://github.com/PiotrMachowski/Home-Assistant-Lovelace-Xiaomi-Vacuum-Map-card/raw/master/dist/textsPL.js) and change filename to `texts.js`

## FAQ
* **Does this card show live map?**
  
  No, you have to prepare your own map in an image
  
* **Does this card require rooted device?**

  No, as it only utilises features already existing in Home Assistant integration
  
* **How to create map?**

  You can use any image you want, the easiest way is to use screenshot from Mi Home/FloleVac.