# STM32G474RE на NUCLEO-G474RE

[До змісту](README.md)

## Вимоги й обмеження пристрою

Для синхронного вимірювання задай джерело тригера, частоту вибірки, допустимі затримку й джитер. Визнач, який стан виходів потрібен після скидання і що має статися, якщо буфер DMA переповнено. [STM32G4 Reference Manual](https://www.st.com/resource/en/reference_manual/rm0440-stm32g4-series-advanced-armbased-32bit-mcus-stmicroelectronics.pdf)

## Цільова платформа

Для першого запуску використовуй вбудований ST-LINK і SWD. Зелений LD2 під’єднано до PA5 за штатного положення SB6. Для власної плати зафіксуй модель MCU, джерело тактування, підключення SWD і карту пам’яті. [Посібник плати NUCLEO-G474RE](https://www.st.com/resource/en/user_manual/um2505-getting-started-with-stm32g4-series-nucleo64-board-stmicroelectronics.pdf), [Апаратна довідка](../wiki/1-hardware-platforms.md#13-stm32)

## Вибір стека прошивки

**STM32CubeG4** дає згенероване налаштування плати та драйвери HAL або LL. Обирай його, коли кілька периферійних блоків потрібно узгодити з тактуванням і виводами. **Bare metal** доречний для вузького набору периферії або власної синхронізації, яку можна перевірити на стенді. [STM32CubeG4](https://github.com/STMicroelectronics/STM32CubeG4), [CMSIS Device G4](https://github.com/STMicroelectronics/cmsis-device-g4)

## Межі для розробника

Тримай тактування, pinout і запуск вимірювань в одному модулі плати. Розробник додає обробку вимірів через наявний інтерфейс буфера. Зміни `.ioc`, linker script, маршруту тригера або налаштувань DMA видно на рев’ю й перевіряються на навантаженому стенді.

## Відтворюване середовище розробки

### Сетап через STM32CubeG4

1. Для Ubuntu 24.04 x86-64 бери STM32CubeMX `6.16.1`, STM32CubeG4 `1.6.1` і GNU toolchain з STM32CubeCLT `1.19.0`. Додай `arm-none-eabi-gcc`, CMake, Ninja і CubeProgrammer з обраного комплекту до `PATH`. У CubeMX вибери `NUCLEO-G474RE`, налаштуй PA5 як push-pull вихід з високим початковим рівнем та згенеруй CMake-проєкт для GNU. Зафіксуй `.ioc`, версію генератора й пакета. [Сумісні пакети для CubeMX 6.16.1](https://www.st.com/resource/en/release_note/rn0094-stm32cubemx-release-6161-stmicroelectronics.pdf), [STM32CubeCLT 1.19.0](https://community.st.com/stm32cubeide-mcus-28/stm32cubeclt-1-19-0-released-155194), [CMake у STM32Cube](https://dev.st.com/stm32cube-docs/stm32cubeide-vscode/latest/en/docs/markup/basic_concepts/cmake.html)
2. З кореня згенерованого проєкту виконай збірку з його `Debug` preset. Перед записом визнач серійний номер ST-LINK у `$PROBE_SN`, а шлях до конкретного ELF — у `$FIRMWARE_ELF`. Після reset має світитися LD2. Для налагодження використовуй конфігурацію ST-LINK у CubeIDE або ST-LINK GDB server і Arm GDB з відповідним ELF. [CubeProgrammer CLI](https://dev.st.com/stm32cube-docs/prog/2.23.0/en/docs/markup/CubeProg_Command_Lines.html), [Налагодження з CubeCLT](https://www.st.com/resource/en/user_manual/um3088-stm32cube-commandline-toolset-quick-start-guide-stmicroelectronics.pdf)

   ```sh
   cmake --preset Debug &&
   cmake --build --preset Debug &&
   STM32_Programmer_CLI -c port=SWD sn="${PROBE_SN:?Вкажи серійний номер ST-LINK}" \
     -w "${FIRMWARE_ELF:?Вкажи ELF поточної збірки}" -v -rst
   ```

3. Тримай прикладні модулі у власних файлах. Для викликів зі згенерованого коду використовуй визначені CubeMX секції `USER CODE`. Перевір регенерацію при зміні `.ioc`, зокрема тактування й призначення виводів. [STM32CubeMX User Manual](https://www.st.com/resource/en/user_manual/um1718-stm32cubemx-for-stm32-configuration-and-initialization-c-code-generation-stmicroelectronics.pdf)

### Сетап без фреймворку

1. Використовуй GNU toolchain із CubeCLT вище. Задай `G4_DEVICE` як шлях до STM32 CMSIS Device G4 `v1.2.5`, `CMSIS_CORE_INCLUDE` як шлях до `CMSIS/Core/Include` у CMSIS_5 `5.9.0`, а `G4_FLASH_LD` як шлях до `STM32G474RETX_FLASH.ld` із шаблону NUCLEO-G474RE у STM32CubeG4 `v1.6.1`. [CMSIS Device G4](https://github.com/STMicroelectronics/cmsis-device-g4/tree/v1.2.5), [CMSIS Core](https://github.com/ARM-software/CMSIS_5/tree/5.9.0/CMSIS/Core), [Linker script для NUCLEO-G474RE](https://github.com/STMicroelectronics/STM32CubeG4/blob/v1.6.1/Projects/NUCLEO-G474RE/Templates/STM32CubeIDE/STM32G474RETX_FLASH.ld)
2. Створи `main.c` з регістровим тестом. Startup викликає `__libc_init_array`. Порожня реалізація підходить для цього C-прикладу без конструкторів. [Register map STM32G4](https://www.st.com/resource/en/reference_manual/rm0440-stm32g4-series-advanced-armbased-32bit-mcus-stmicroelectronics.pdf), [Startup STM32G474](https://github.com/STMicroelectronics/cmsis-device-g4/blob/v1.2.5/Source/Templates/gcc/startup_stm32g474xx.s)

   ```c
   #include "stm32g4xx.h"

   void __libc_init_array(void) {}

   int main(void)
   {
       RCC->AHB2ENR |= RCC_AHB2ENR_GPIOAEN;
       (void)RCC->AHB2ENR;
       GPIOA->BSRR = 1u << 5;
       GPIOA->OTYPER &= ~(1u << 5);
       GPIOA->MODER = (GPIOA->MODER & ~(3u << 10)) | (1u << 10);
       for (;;) {}
   }
   ```

3. Збери з каталогу `main.c`. Використай один ABI для всіх файлів. Після успішної збірки запиши отриманий `build/g474-register/firmware.elf` через CubeProgrammer за параметрами SDK-маршруту.

   ```sh
   mkdir -p build/g474-register &&
   arm-none-eabi-gcc -mcpu=cortex-m4 -mthumb -mfloat-abi=soft -Os -g -ffreestanding \
     -DSTM32G474xx -I"${G4_DEVICE:?}/Include" -I"${CMSIS_CORE_INCLUDE:?}" \
     -T "${G4_FLASH_LD:?}" -nostartfiles --specs=nosys.specs \
     -Wl,-Map=build/g474-register/firmware.map -o build/g474-register/firmware.elf \
     "$G4_DEVICE/Source/Templates/gcc/startup_stm32g474xx.s" \
     "$G4_DEVICE/Source/Templates/system_stm32g4xx.c" main.c
   ```

4. Для покрокового сеансу запусти `ST-LINK_gdbserver -d -i "$PROBE_SN" -cp "$CUBEPROGRAMMER_BIN"`, де `CUBEPROGRAMMER_BIN` вказує на `bin` із CubeProgrammer. Відкрий той самий ELF в `arm-none-eabi-gdb` і виконай `target remote localhost:61234` на типовому порті сервера. Постав точки зупину в `Reset_Handler` і `main`, перевір вектори, стек, `.data` і `.bss`. Після холодного ввімкнення має світитися LD2. [Параметри ST-LINK GDB Server](https://www.st.com/resource/en/user_manual/dm00613038-stm32cubeide-st-link-gdb-server-stmicroelectronics.pdf)

## Перевірка й фіксація рішення

Для виробу із синхронними вимірюваннями перевір на платі ланцюг таймер → ADC → DMA. Запиши умови тактування, частоту вибірки, затримку та джитер при робочому навантаженні. Для виробничого образу зафіксуй політику option bytes та ділянки, які мають переживати оновлення. [STM32G4 Reference Manual](https://www.st.com/resource/en/reference_manual/rm0440-stm32g4-series-advanced-armbased-32bit-mcus-stmicroelectronics.pdf), [CubeProgrammer CLI](https://dev.st.com/stm32cube-docs/prog/2.23.0/en/docs/markup/CubeProg_Command_Lines.html)

Зафіксуй обраний стек, конфігурацію тактування й виміряні межі, які визначили вибір HAL, LL або регістрового доступу.
