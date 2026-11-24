# ESP32-C3 та ESP32-S3

[До змісту](README.md)

## Вимоги й обмеження пристрою

Для мережевого вузла визнач, що він робить після обриву Wi-Fi, скільки часу має на відновлення з’єднання та які налаштування зберігає після перезапуску. Зафіксуй бюджети Flash, RAM, PSRAM і пікового струму. Якщо потрібне оновлення в полі, опиши поведінку після перерваного запису або невдалого старту. [Поради для апаратного дизайну ESP32-C3](https://docs.espressif.com/projects/esp-hardware-design-guidelines/en/latest/esp32c3/schematic-checklist.html), [OTA в ESP-IDF](https://docs.espressif.com/projects/esp-idf/en/stable/esp32c3/api-reference/system/ota.html)

## Цільова платформа

Для C3 використовуй **ESP32-C3-DevKitM-1** і її порт USB–UART. Вбудований RGB LED під’єднано до GPIO8. Для S3 використовуй **Waveshare ESP32-S3-Zero** з 4 МБ Flash і USB Serial/JTAG. Вбудований RGB LED під’єднано до GPIO21. Для запису задай `$PORT` для однієї підключеної тестової плати. [ESP32-C3-DevKitM-1](https://docs.espressif.com/projects/esp-dev-kits/en/latest/esp32c3/esp32-c3-devkitm-1/user_guide.html), [Схема ESP32-S3-Zero](https://files.waveshare.com/wiki/ESP32-S3-Zero/ESP32-S3-Zero-Sch.pdf), [Апаратна довідка](../wiki/1-hardware-platforms.md#12-esp32)

## Вибір стека прошивки

**ESP-IDF** підходить для пристрою з Wi-Fi, Bluetooth LE або OTA, коли потрібні готові стеки та підтримуваний процес оновлення. **Bare metal** підходить для локального GPIO, таймера або UART, коли команда готова підтримувати власний старт і формат образу. [Wi-Fi ESP-IDF](https://docs.espressif.com/projects/esp-idf/en/stable/esp32c3/api-reference/network/esp_wifi.html), [Формат образу ESP32](https://docs.espressif.com/projects/esptool/en/latest/esp32c3/advanced-topics/firmware-image-format.html)

## Межі для розробника

Признач власника цільової конфігурації, таблиці розділів або власного linker script. Розробник змінює поведінку в застосунку та збирає ціль штатною командою. Зміни адрес, типу модуля або режиму Flash видно в diff і перевіряються окремою збіркою та холодним стартом відповідної плати.

## Відтворюване середовище розробки

### Сетап через ESP-IDF

1. Для Ubuntu 24.04 x86-64 бери ESP-IDF `v5.4.2` і GCC `esp-14.2.0_20241119` з комплекту цього тегу. Склонуй тег із підмодулями. У корені ESP-IDF виконай `./install.sh esp32c3,esp32s3`, потім `source export.sh`. Скопіюй `examples/get-started/blink` в окремий каталог для кожної цілі. [Встановлення ESP-IDF 5.4.2](https://docs.espressif.com/projects/esp-idf/en/v5.4.2/esp32c3/get-started/linux-macos-setup.html), [Маніфест інструментів](https://github.com/espressif/esp-idf/blob/v5.4.2/tools/tools.json)
2. У каталозі прикладу задай `$CHIP` як `esp32c3` або `esp32s3`. Після `set-target` у `menuconfig` обери адресний LED із RMT, GPIO8 для C3 або GPIO21 для S3. Після запису очікуй блимання. Для покрокового налагодження S3 під’єднай USB Serial/JTAG і запусти `idf.py openocd gdb`. Для C3 використовуй підтримуване JTAG-підключення за документацією плати. [Blink для ESP-IDF 5.4.2](https://github.com/espressif/esp-idf/tree/v5.4.2/examples/get-started/blink), [JTAG для C3](https://docs.espressif.com/projects/esp-idf/en/v5.4.2/esp32c3/api-guides/jtag-debugging/using-debugger.html), [USB Serial/JTAG для S3](https://docs.espressif.com/projects/esp-idf/en/v5.4.2/esp32s3/api-guides/jtag-debugging/configure-builtin-jtag.html)

   ```sh
   idf.py set-target "${CHIP:?Вкажи esp32c3 або esp32s3}" &&
   idf.py menuconfig &&
   idf.py build &&
   idf.py -p "${PORT:?Вкажи порт тестової плати}" flash
   ```

3. Перед додаванням функцій виробу зафіксуй `sdkconfig.defaults`, конфігурації C3/S3, CSV таблиці розділів, `idf_component.yml` та lock-файл за правилами Component Manager. Для виробництва бери bootloader, таблицю розділів, застосунок і адреси запису з конкретної збірки. Прив’яжи ELF для декодування журналу до того самого випуску. [Таблиця розділів](https://docs.espressif.com/projects/esp-idf/en/stable/esp32c3/api-guides/partition-tables.html), [Lock-файл залежностей](https://docs.espressif.com/projects/idf-component-manager/en/latest/reference/dependencies_lock.html)

### Сетап без фреймворку

Для цих прикладів використовуй виділену плату з вимкненими Secure Boot і Flash Encryption та доступним ROM download mode. Запис за `0x0` замінює чинний завантажувач. Перед записом збережи повний відновлюваний набір образів і їхні адреси. У разі збою переведи плату в ROM download mode через BOOT і RESET та запиши цей набір. [Secure Boot v2](https://docs.espressif.com/projects/esp-idf/en/v5.4.2/esp32s3/security/secure-boot-v2.html), [Режим завантаження C3](https://docs.espressif.com/projects/esptool/en/latest/esp32c3/advanced-topics/boot-mode-selection.html), [Режим завантаження S3](https://docs.espressif.com/projects/esptool/en/latest/esp32s3/advanced-topics/boot-mode-selection.html)

**ESP32-C3.** На чипі ревізії 3 або новішій зі станом eFuse, що дозволяє direct boot, використовуй приклад Espressif з власними startup і linker script. Потрібні xPack RISC-V GCC `12.2.0-3`, CMake, Ninja та `esptool` 5.x. У checkout нижче під’єднай зовнішній LED із резистором між GPIO2 і 3V3 за схемою прикладу. Результат збірки — ELF `blink`, карта `blink.map` і `blink.bin` у `build/esp32c3-direct/`. Після холодного старту очікуй блимання зовнішнього LED. [Direct boot і його передумови](https://github.com/espressif/esp32c3-direct-boot-example/tree/bac08da009a120fbca7f6b290d270fd5bf4df819), [Схема та збірка Blink](https://github.com/espressif/esp32c3-direct-boot-example/tree/bac08da009a120fbca7f6b290d270fd5bf4df819/examples/blink)

```sh
git clone https://github.com/espressif/esp32c3-direct-boot-example.git esp32c3-direct &&
git -C esp32c3-direct checkout --detach bac08da009a120fbca7f6b290d270fd5bf4df819 &&
cmake -S esp32c3-direct/examples/blink -B build/esp32c3-direct -Dtarget=esp32c3 -G Ninja &&
cmake --build build/esp32c3-direct &&
esptool --chip esp32c3 --port "${PORT:?Вкажи порт C3}" write-flash 0x0 build/esp32c3-direct/blink.bin
```

**ESP32-S3.** На ESP32-S3-Zero використовуй проєкт `gpio_button` з власним startup, linker script і регістровими драйверами. Додай `xtensa-esp32s3-elf-gcc` з релізу Espressif `esp-14.2.0_20241119`, `esptool` 5.x і GNU Make до `PATH`. Проєкт створює `build/gpio_button.elf` та ROM-образ `build/gpio_button.bin`. Після запуску натискання BOOT має вмикати зелений LED, відпускання — вимикати. [Реліз GCC](https://github.com/espressif/crosstool-NG/releases/tag/esp-14.2.0_20241119), [Проєкт для ESP32-S3-Zero](https://github.com/codewithamoako/esp32s3-baremetal/tree/df1798fc09eb9232b690aa417f4fae2892cadf1130), [Вхід і вихід прикладу](https://github.com/codewithamoako/esp32s3-baremetal/blob/df1798fc09eb9232b690aa417f4fae2892cadf1130/examples/gpio_button.c)

```sh
git clone https://github.com/codewithamoako/esp32s3-baremetal.git esp32s3-zero &&
git -C esp32s3-zero checkout --detach df1798fc09eb9232b690aa417f4fae2892cadf1130 &&
make -C esp32s3-zero ESPTOOL=esptool gpio_button &&
esptool --chip esp32s3 --port "${PORT:?Вкажи порт S3}" write-flash 0x0 esp32s3-zero/build/gpio_button.bin
```

## Перевірка й фіксація рішення

Збирай C3 і S3 окремими цілями в CI. Для продукту з OTA перевір завантаження фінального образу, підтвердження його працездатності та відкат у разі збою. Для мережевого виробу додай повторне з’єднання й пусковий профіль живлення на цільовому модулі. У ESP-IDF для стендових перевірок можна використати `pytest-embedded`. [Тести ESP-IDF](https://docs.espressif.com/projects/esp-idf/en/stable/esp32/contribute/esp-idf-tests-with-pytest.html), [OTA rollback](https://docs.espressif.com/projects/esp-idf/en/stable/esp32c3/api-reference/system/ota.html)

Зафіксуй вибір стека разом із моделлю модуля, схемою розділів або форматом власного образу та результатом холодного запуску.
