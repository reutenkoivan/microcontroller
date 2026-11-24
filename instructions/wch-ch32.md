# WCH CH32V003, CH32V203 та CH32F203

[До змісту](README.md)

## Вимоги й обмеження пристрою

Визнач потрібні виводи й допустиму затримку обробки входу. Для V003 задай бюджет Flash, RAM і стека та залиш доступ до SDI. Якщо ціль використовує USB або CAN, опиши реальний хост чи трансивер для перевірки. [CH32V003 EVT](https://github.com/openwch/ch32v003), [CH32V20x EVT](https://github.com/openwch/ch32v20x)

## Цільова платформа

Тестові цілі — WCH CH32V003F4 EVT з виходом PD0, CH32V203C8 EVT з PA0 і CH32F203C8T6 EVT з PA0. Вимірюй сигнал між тестовим виводом і GND. Для V003 використовуй WCH-LinkE у режимі RISC-V через SDI (PD1), для V203 — той самий режим і двопровідний інтерфейс за схемою EVT. Для F203 перемкни WCH-LinkE у режим Arm і під’єднай SWD. [CH32V003 EVT](https://github.com/openwch/ch32v003), [CH32V20x EVT](https://github.com/openwch/ch32v20x), [CH32F20x EVT](https://github.com/openwch/ch32f20x), [WCH-Link User Manual](https://www.wch.cn/uploads/file/20250124/1737704462135866.pdf), [Апаратна довідка](../wiki/1-hardware-platforms.md#110-wch-ch32)

## Вибір стека прошивки

**WCH EVT** дає стартові проєкти й бібліотеки для конкретної серії. Для підтримуваних CH32V можна обрати WCH Arduino Core, якщо його бібліотеки покривають сценарій. **Bare metal** підходить для невеликого набору периферії, коли команда підтримує startup, карту пам’яті та параметри ISA. [WCH Arduino Core](https://github.com/openwch/arduino_core_ch32), [CH32F20x EVT](https://www.wch.cn/downloads/CH32F20xEVT_ZIP.html)

## Межі для розробника

Закріпи параметри MCU й схему доступу для налагодження у цілі плати. Розробник використовує вільний GPIO або наявний інтерфейс шини. Зміни ISA, startup, linker script чи призначення debug-виводів перевіряй за diff та фактичним підключенням програматора.

## Відтворюване середовище розробки

### Сетап через WCH Arduino Core або EVT

1. **CH32V003 / CH32V203.** На Ubuntu 24.04 x86-64 встанови Arduino CLI 1.3.1. Використовуй WCH Arduino Core 1.0.5 із закріпленого індексу WCH. Цей Core завантажує WCH GCC 15.2.0 і WCH OpenOCD 2.10.0. Створи `GpioToggle/GpioToggle.ino`, де `setup()` налаштовує `PD0` для V003 або `PA0` для V203 через `pinMode(pin, OUTPUT)`. У `loop()` чергуй `digitalWrite(pin, HIGH)` і `digitalWrite(pin, LOW)` із `delay(500)` після кожного запису. Задай `FQBN` як `WCH:ch32v:CH32V00x_EVT:pnum=CH32V003F4` або `WCH:ch32v:CH32V20x_EVT:pnum=CH32V203C8` відповідно. Для кожної плати використовуй окремий робочий каталог. [Закріплений індекс WCH](https://github.com/openwch/board_manager_files/blob/1ca9500eef50846b88af4a638a9bbc308b5d4b0d/package_ch32v_index.json), [Профілі WCH Arduino Core 1.0.5](https://github.com/openwch/arduino_core_ch32/blob/1.0.5/boards.txt)

   ```sh
   WCH_INDEX=https://raw.githubusercontent.com/openwch/board_manager_files/1ca9500eef50846b88af4a638a9bbc308b5d4b0d/package_ch32v_index.json
   arduino-cli core update-index --additional-urls "$WCH_INDEX" &&
   arduino-cli core install WCH:ch32v@1.0.5 --additional-urls "$WCH_INDEX" &&
   arduino-cli compile --additional-urls "$WCH_INDEX" --fqbn "${FQBN:?Set the board FQBN}" --output-dir build/arduino GpioToggle
   ```

2. На стенді з однією підключеною RISC-V платою додай `bin` із WCH OpenOCD до `PATH` і задай `WCH_OPENOCD_CFG` як шлях до `wch-riscv.cfg` із цього самого пакета. Запиши свіжий `build/arduino/GpioToggle.ino.elf` командою нижче. Для debug запусти OpenOCD з тим самим конфігом без команди запису та під’єднай WCH GDB з ELF. [Рецепт OpenOCD і конфігурація debug](https://github.com/openwch/arduino_core_ch32/blob/1.0.5/platform.txt)

   ```sh
   openocd -f "${WCH_OPENOCD_CFG:?Set the WCH OpenOCD configuration}" -c init -c halt \
     -c 'program {build/arduino/GpioToggle.ino.elf} verify; wlink_reset_resume; exit;'
   ```

3. **CH32F203.** На Ubuntu 24.04 x86-64 встанови Arm GNU Toolchain 13.3.Rel1 і GNU Make. Отримай порт бібліотеки EVT на GNU Make за ревізією `30490d016d607db81800f0a8592af72415d63546` і виконай `make` у його корені. Отриманий `build/ch32f203c8t6.hex` запиши та перевір через WCH-LinkUtility на Windows із WCH-LinkE в режимі Arm і SWD. Після перезапуску перевір перемикання PA0. Для debug використовуй `build/ch32f203c8t6.elf` у MounRiver Studio з налаштуванням Arm SWD для CH32F203C8T6. [Закріплений EVT-проєкт для GNU Make](https://github.com/cjacker/ch32f20xevt_gcc_makefile/tree/30490d016d607db81800f0a8592af72415d63546), [Arm GNU Toolchain 13.3.Rel1](https://developer.arm.com/downloads/-/arm-gnu-toolchain-downloads/13-3-rel1), [WCH-Link](https://www.wch.cn/products/wch-link)

### Сетап без фреймворку

1. Використовуй компілятори з попереднього маршруту. Для V003 і V203 додай до `PATH` каталог `packages/WCH/tools/riscv-wch-gcc/15.2.0/bin` у каталозі даних Arduino CLI (типово `~/.arduino15` на Linux). З репозиторіїв нижче бери лише startup і linker script. Задай `V003_SRC` і `V203_SRC` як шляхи до відповідного `EVT/EXAM/SRC`, а `F203_SRC` — до `CH32F_firmware_library` у закріпленому EVT-порті. [V003 EVT, ревізія 4e1b743](https://github.com/openwch/ch32v003/tree/4e1b7437ada5d3ae8831a6f4b811a6320e4e23c7/EVT/EXAM/SRC), [V203 EVT, ревізія b375bbb](https://github.com/openwch/ch32v20x/tree/b375bbbc7dfe8642d6fb7c17e6d3207ee1c3c9c1/EVT/EXAM/SRC), [F203 startup і linker](https://github.com/cjacker/ch32f20xevt_gcc_makefile/tree/30490d016d607db81800f0a8592af72415d63546/CH32F_firmware_library)

2. Створи `main.c` з кодом перевірки GPIO. Для цього тесту збережи тактування після reset, а для C-образу F203 надай порожній виклик ініціалізації конструкторів, який очікує startup. [CH32V003 Reference Manual](https://www.wch.cn/downloads/CH32V003RM_PDF.html), [CH32V20x Reference Manual](https://www.wch.cn/downloads/CH32V20xRM_PDF.html), [CH32F203 Reference Manual](https://www.wch.cn/uploads/file/20230404/1680573065146901.pdf)

   ```c
   #include <stdint.h>

   #define REG32(address) (*(volatile uint32_t *)(address))
   #define RCC_APB2PCENR REG32(0x40021018u)

   #if defined(TEST_PD0)
   #define GPIO_BASE 0x40011400u
   #define GPIO_CLOCK (1u << 5)
   #else
   #define GPIO_BASE 0x40010800u
   #define GPIO_CLOCK (1u << 2)
   #endif

   void SystemInit(void) {}

   #if defined(TEST_F203)
   void __libc_init_array(void) {}
   #endif

   int main(void)
   {
       RCC_APB2PCENR |= GPIO_CLOCK;
       REG32(GPIO_BASE + 0x10u) = 1u;
       REG32(GPIO_BASE) = (REG32(GPIO_BASE) & ~0xFu) | 0x2u;
       for (;;) {}
   }
   ```

3. Виконай лише команду для вибраної плати з каталогу `main.c`. Перевір карту ELF та вибір адаптера перед записом. Запис для V003 і V203 виконай тією самою командою OpenOCD, підставивши відповідний свіжий ELF. Для F203 створи HEX через `arm-none-eabi-objcopy -O ihex f203.elf f203.hex` після успішної збірки й запиши через WCH-LinkUtility. Після холодного старту на PD0 або PA0 має бути високий рівень. [Параметри ISA та ABI у профілях виробника](https://github.com/openwch/arduino_core_ch32/blob/1.0.5/boards.txt)

   ```sh
   riscv32-wch-elf-gcc -march=rv32ecxw -mabi=ilp32e -DTEST_PD0 \
     -ffreestanding -nostdlib -Os -g -T "${V003_SRC:?}/Ld/Link.ld" \
     -Wl,-Map,v003.map -o v003.elf "$V003_SRC/Startup/startup_ch32v00x.S" main.c
   ```

   ```sh
   riscv32-wch-elf-gcc -march=rv32imacxw -mabi=ilp32 \
     -ffreestanding -nostdlib -Os -g -T "${V203_SRC:?}/Ld/Link.ld" \
     -Wl,-Map,v203.map -o v203.elf "$V203_SRC/Startup/startup_ch32v20x_D6.S" main.c
   ```

   ```sh
   arm-none-eabi-gcc -mcpu=cortex-m3 -mthumb -DTEST_F203 \
     -ffreestanding -nostdlib -Os -g -T "${F203_SRC:?}/Ld/Link.ld" \
     -Wl,-Map,f203.map -o f203.elf "$F203_SRC/Startup/startup_ch32f20x_D6.S" main.c
   ```

## Перевірка й фіксація рішення

На V003 стеж за бюджетом стека й доступністю лінії SDI, якщо застосунок змінює призначення її виводу. Для V203 і F203 перевір USB або CAN із реальним хостом чи трансивером, якщо інтерфейс входить у контракт виробу. Для кожної плати відпрацюй її документований режим відновлення після помилки запуску й перевір підключення програматора. [WCH-Link User Manual](https://www.wch.cn/uploads/file/20250124/1737704462135866.pdf), [CH32V20x EVT](https://github.com/openwch/ch32v20x), [CH32F203](https://www.wch.cn/products/CH32F203.html)

Запиши обраний стек, параметри цільового чипа та підтверджений спосіб запису й відновлення для ревізії плати.
