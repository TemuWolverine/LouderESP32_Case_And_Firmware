## Watch the video

[![Watch The Video](https://img.youtube.com/vi/cPmEKW6hQlM/0.jpg)](https://www.youtube.com/watch?v=cPmEKW6hQlME)


## Parts List

| Part | Quantity     | Link |
|------|-------------|-------|
| LouderESP S3 | 1 | https://www.tindie.com/products/sonocotta/louder-esp32/ |
| Binding posts | 4 | https://s.click.aliexpress.com/e/_c35gjqbZ |
| USB-PD decoy board  | 1 | https://s.click.aliexpress.com/e/_c3sX2BbH |
| Buck Converter | 1 | https://s.click.aliexpress.com/e/_c36jU1IT |
| WS2812 144LED/m| 1 | https://s.click.aliexpress.com/e/_c3ujoDZD |
| Rotary Encoder | 1 | |
| Brass (plated) knob | 1 | https://www.bunnings.com.au/taskmaster-32-5mm-knurled-brushed-brass-cabinet-knobs-4-pack_p0651903 |
| M2.5 Heatset Insert     | 11 |       |
| M2.5 x 8mm screws     | 11 |       |

### Filament
If you want to use the exact same filament as I did in the video...
* [Filament Hub Matte Dark Green Forest](https://www.filamenthub.com.au/products/pla-matte-dark-forest-green)
* [Bambulab PLA Matte Bone White](https://au.store.bambulab.com/products/pla-matte?id=563873274318602240)

## Firmware
This firmware is more or less a copy-paste of [Sonocotta's Louder-ESP32 sendspin firmware](https://github.com/sonocotta/esp32-audio-dock/blob/main/firmware/esphome/7-louder-esp32-plus/louder-esp32-plus-idf-sendspin.yaml).

There are some new variables for the pins being used

```yaml
#==================tmuWolverine Changes
volume_led_pin: GPIO05

#volume rotary encoder
rotary_pin_a: GPIO10
rotary_pin_b: GPIO12
rotary_pin_button: GPIO6
```
As well as code for the volume LEDs to show the current volume, and the rotary encoder to manipulate the volume

```yaml
#==================tmuWolverine Changes
light:
  - platform: esp32_rmt_led_strip
    name: volumeLED
    rgb_order: GRB
    chipset: WS2812
    pin: ${volume_led_pin}
    num_leds: 5
    id: volumeled
    rmt_symbols: 48

    effects:
      - addressable_lambda:
          name: "Volume Display"
          update_interval: 50ms
          lambda: |-
            int leds = (int) ceil(id(current_volume) * 5.0f);

            for (int i = 0; i < 5; i++) {
              if (i < leds) {
                it[i] = Color(0, 255, 0);
              } else {
                it[i] = Color(0, 0, 0);
              }
            }

globals:
  - id: last_volume
    type: float
    initial_value: '-1.0'

script:
  - id: update_volume_leds
    then:
      - light.turn_on:
          id: volumeled
          effect: "Volume Display"

interval:
  - interval: 50ms
    then:
      - if:
          condition:
            lambda: |-
              return id(current_volume) != id(last_volume);
          then:
            - lambda: |-
                id(last_volume) = id(current_volume);
            - script.execute: update_volume_leds
                        
sensor:
- platform: rotary_encoder
  name: "Rotary Encoder"
  pin_a: ${rotary_pin_b}
  pin_b: ${rotary_pin_a}
  on_clockwise:
    - lambda: |-
        float new_vol = id(current_volume) + 0.05f;
        if (new_vol > 1.0f) 
          new_vol = 1.0f;
        id(current_volume) = new_vol;
    - media_player.volume_set:
        id: external_media_player
        volume: !lambda "return id(current_volume);"

  on_anticlockwise:
    - lambda: |-
        float new_vol = id(current_volume) - 0.05f;
        if (new_vol < 0.0f)
          new_vol = 0.0f;
        id(current_volume) = new_vol;
    - media_player.volume_set:
        id: external_media_player
        volume: !lambda "return id(current_volume);"
```
