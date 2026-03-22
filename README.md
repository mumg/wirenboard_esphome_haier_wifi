# Haier ESPHome MQTT

ESPHome-проект для управления кондиционером Haier через UART и MQTT без Home Assistant API.

## Что делает проект

- Подключает ESP32 к кондиционеру Haier по UART.
- Управляет кондиционером через MQTT-команды.
- Публикует метаданные устройства и контролов в MQTT.
- Публикует текущее состояние кондиционера каждые `15s`.
- Поддерживает OTA-обновление прошивки.

## Требования

- ESP32 `esp32doit-devkit-v1`
- установленный `esphome`
- MQTT-брокер
- подключение UART к плате кондиционера Haier

## Настройка

В проекте используется файл `secrets.yaml`. В нем должны быть определены:

```yaml
wifi_ssid: "YOUR_WIFI_SSID"
wifi_password: "YOUR_WIFI_PASSWORD"
mqtt_host: "192.168.1.10"
mqtt_port: "1883"
mqtt_username: "mqtt_user"
mqtt_password: "mqtt_password"
ota_password: "change-me"
```

При необходимости поменяй в [wirenboard_haier_wifi.yml](/Users/maxim/work/home/haier_esphome/wirenboard_haier_wifi.yml):

- `device_name`
- `friendly_name`
- `haier_uart_tx_pin`
- `haier_uart_rx_pin`
- `esp32.board`

## Сборка

Проверка конфига:

```bash
esphome config wirenboard_haier_wifi.yml
```

Сборка прошивки:

```bash
esphome compile wirenboard_haier_wifi.yml
```

Первая прошивка по USB:

```bash
esphome run wirenboard_haier_wifi.yml
```

Обновление по сети:

```bash
esphome upload wirenboard_haier_wifi.yml
```

Логи:

```bash
esphome logs wirenboard_haier_wifi.yml
```

## MQTT устройство

После подключения публикуются retained-топики устройства:

- `/devices/haier-${device_name}`
- `/devices/haier-${device_name}/meta`
- `/devices/haier-${device_name}/meta/driver`
- `/devices/haier-${device_name}/meta/name`

Также публикуются retained `meta`-описания для контролов:

- `power`
- `connected`
- `mode`
- `preset`
- `fan`
- `fan_speed`
- `ionization`
- `vertical_swing`
- `horizontal_swing`
- `swing`
- `current_temperature`
- `target_temperature`

## MQTT команды

Входящие команды отправляются в топики `/devices/haier-${device_name}/controls/<control>/on`.

Поддерживаются:

- `power/on`
  - `1` -> включить
  - `0` -> выключить
- `mode/on`
  - `1` -> heat
  - `2` -> cool
  - `3` -> heat_cool
  - `4` -> dry
  - `5` -> fan_only
- `preset/on`
  - `1` -> away
  - `2` -> boost
  - `3` -> comfort
  - любое другое значение -> none
- `fan/on`
  - `1` -> auto
  - `2` -> low
  - `3` -> medium
  - `4` -> high
- `fan_speed/on`
  - `1` -> low
  - `2` -> medium
  - `3` -> high
  - любое другое значение -> auto
- `ionization/on`
  - `1` -> on
  - `0` -> off
- `vertical_swing/on`
  - `0` -> auto
  - `1..7` -> фиксированные положения
- `horizontal_swing/on`
  - `0` -> auto
  - `1..5` -> фиксированные положения
- `swing/on`
  - `0` -> off
  - `1` -> both
  - `2` -> horizontal
  - `3` -> vertical
- `target_temperature/on`
  - целое число температуры

Пример:

```bash
mosquitto_pub -h 192.168.1.10 -t /devices/haier-haier-mqtt-node/controls/power/on -m 1
```

## MQTT состояния

Публикуются текущие значения в топики `/devices/haier-${device_name}/controls/<control>`.

Основные публикуемые состояния:

- `connected`
- `ionization`
- `preset`
- `fan`
- `mode`
- `vertical_swing`
- `horizontal_swing`
- `swing`
- `current_temperature`
- `target_temperature`

Шаблонные сенсоры обновляются каждые `15s`.

## Ограничения

- README описывает текущую логику из конфига, но не заменяет проверку через `esphome config`.
- Для успешной сборки важна совместимость используемой версии ESPHome с компонентом `haier`.
