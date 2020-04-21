
# Простой эмулятор 8/16 битного процессора и компилятор к нему

**0x64 cm)** - "Простой" эмулятор 8/16 битного процессора (не какого-то конкретного). Он имеет простенький сет инструкций, 2 кб ОЗУ, 4 8/16-битных регистра общего назначения, указатель стэка(не используется), счетчик программы.

**ASMC** - "Простой" компилятор для 0x64 cm) использует синтаксис Intel, не чувствительный к регистру. На вход берет файл с интеловским ассемблером а на выходе выдает бинарник который можно запустить на эмуляторе.

* [Процессор](#Процессор)
  * [Регистры](#Регистры)
  * [Сет инструкций](#Сет-инструкций)
  * [Консольное меню](#Консольное-меню)
  * [Выполнение программы](#Выполнение-программы)
* [Компилятор](#Компилятор)
  * [Синтаксис](#Синтаксис)
  * [Бинарные файлы](#Бинарные-файлы)
  * [Параметры командной строки](#Параметры-командной-строки)
* [FAQ](#FAQ)
  * [В планах](#В-планах)


## Процессор

### Регистры

Все регистры(кроме статус-регистра) имеют следующую структуру:

|  Регистр AX (16 бит) |
| -- |
| *Старшие 8 бит* |
| *Регистр A (младшие 8 бит)* |

Размер регистра AX 16 бит, младшие 8 из которых занимает регистр A

### Сет инструкций

|   0x  |      0         |        1         |       2        |      3         |        4         |       5        |      6         |        7         |       8        |      9      |      A     |      6      |     7      |     8      |      9      |      A      |
|   --  |      --        |       --         |       --       |      --        |       --         |       --       |      --        |       --         |       --       |      --     |     --     |     --      |     --     |     --     |     --      |      --     |
| **0** |`MOV R, d8`     |`MOV [R], d8`     |`MOV a16, d8`   |`ADD R, d8`     |`ADD [R], d8`     |`ADD a16, d8`   |`SUB R, d8`     |`SUB [R], d8`     |`SUB a16, d8`   |`PUSH d8`    |            |             |            |            |             |`NOP`        |
| **1** |`MOV R, R`      |`MOV [R], R`      |`MOV a16, R`    |`ADD R, R`      |`ADD [R], R`      |`ADD a16, R`    |`SUB R, R`      |`SUB [R], R`      |`SUB a16, R`    |`PUSH R`     |`POP R`     |             |            |            |             |             |
| **2** |`MOV R, a16`    |`MOV [R], a16`    |`MOV b a16, a16`|`ADD R, a16`    |`ADD [R], a16`    |`ADD b a16, a16`|`SUB R, a16`    |`SUB [R], a16`    |`SUB b a16, a16`|`PUSH a16`   |`POP a16`   |             |            |            |             |             |
| **3** |`MOV R, [R]`    |`MOV [R], [R]`    |`MOV a16, [R]`  |`ADD R, [R]`    |`ADD [R], [R]`    |`ADD a16, [R]`  |`SUB R, [R]`    |`SUB [R], [R]`    |`SUB a16, [R]`  |`PUSH [R]`   |`POP [R]`   |             |            |            |             |             |
| **4** |`MOV R16, d16`  |`MOV [R16], d16`  |`MOV a16, d16`  |`ADD R16, d16`  |`ADD [R16], d16`  |`ADD a16, d16`  |`SUB R16, d16`  |`SUB [R16], d16`  |`SUB a16, d16`  |`PUSH d16`   |            |             |            |            |             |             |
| **5** |`MOV R16, R16`  |`MOV [R16], R16`  |`MOV a16, R16`  |`ADD R16, R16`  |`ADD [R16], R16`  |`ADD a16, R16`  |`SUB R16, R16`  |`SUB [R16], R16`  |`SUB a16, R16`  |`PUSH R16`   |`POP R16`   |             |            |            |             |             |
| **6** |`MOV R16, a16`  |`MOV [R16], a16`  |`MOV w a16, a16`|`ADD R16, a16`  |`ADD [R16], a16`  |`ADD w a16, a16`|`SUB R16, a16`  |`SUB [R16], a16`  |`SUB w a16, a16`|`PUSH a16`   |`POP a16`   |             |            |            |             |             |
| **7** |`MOV R16, [R16]`|`MOV [R16], [R16]`|`MOV a16, [R16]`|`ADD R16, [R16]`|`ADD [R16], [R16]`|`ADD a16, [R16]`|`SUB R16, [R16]`|`SUB [R16], [R16]`|`SUB a16, [R16]`|`PUSH [R16]` |`POP [R16]` |             |            |            |             |             |
| **8** |`CMP R, d8`     |`CMP [R], d8`     |`CMP a16, d8`   |                |                  |                |                |                  |                |             |            |             |            |            |             |             |
| **9** |`CMP R, R`      |`CMP [R], R`      |`CMP a16, R`    |                |                  |                |                |                  |                |             |            |             |            |            |             |             |
| **A** |`CMP R, a16`    |`CMP [R], a16`    |`CMP b a16, a16`|                |                  |                |                |                  |                |             |            |             |            |            |             |`HLT`        |
| **B** |`CMP R, [R]`    |`CMP [R], [R]`    |`CMP a16, [R]`  |                |                  |                |                |                  |                |             |            |             |            |            |             |             |
| **C** |`CMP R16, d16`  |`CMP [R16], d16`  |`CMP a16, d16`  |                |                  |                |                |                  |                |`JMP a16`    |`JE a16`    |`JNE a16`    |`JL a16`    |`JG a16`    |`JLE a16`    |`JGE a16`    |
| **D** |`CMP R16, R16`  |`CMP [R16], R16`  |`CMP a16, R16`  |                |                  |                |                |                  |                |`JMP R/R16`  |`JE R/R16`  |`JNE R/R16`  |`JL R/R16`  |`JG R/R16`  |`JLE R/R16`  |`JGE R/R16`  |
| **E** |`CMP R16, a16`  |`CMP [R16], a16`  |`CMP w a16, a16`|                |                  |                |                |                  |                |`JMP [a16]`  |`JE [a16]`  |`JNE [a16]`  |`JL [a16]`  |`JG [a16]`  |`JLE [a16]`  |`JGE [a16]`  |
| **F** |`CMP R16, [R16]`|`CMP [R16], [R16]`|`CMP a16, [R16]`|                |                  |                |                |                  |                |`JMP [R/R16]`|`JE [R/R16]`|`JNE [R/R16]`|`JL [R/R16]`|`JG [R/R16]`|`JLE [R/R16]`|`JGE [R/R16]`|

---

![Сет инструкций](https://i.ibb.co/jbbcRLx/instset.png)

---

| Сигнатура |  Значение    |
|    --     |      --      |
|  **R**    | 8 битный регистр |
|  **R16**  | 16 битный регистр |
|  **d8**   | 8 битное число |
|  **d16**  | 16 битное число |
|  **a16**  | 16 битный адрес |
|    --     |      --      |
|  **w (word)**  | 16 битный тип данных |
|  **b (byte)**  | 8 битный тип данных |

---

### Консольное меню

Для удобства работы с эмулятором и его дебагинга было добавлено консольное меню

| Команда |  Описание    |
|          --             |      --      |
| **.registers, .regs**   | При вводе этой команды в консоль выведется содержимое регистров и флагов|
| **.memory, .mem**       | При вводе этой команды в консоль выведется содержимое ВСЕХ 2 кб ОЗУ|
| **.execute, .exec**     | При вводе этой команды процессор начнет работать в режиме выполнения до команды HALT (не будет спрашивать команду)|
| **.reset**              | При вводе этой команды произойдет перезапуск|
| **.disassemble, .dasm** | При вводе этой команды в консоль выведется дизассемблированые первие 21 байт памяти|
| **.exit**               | При вводе этой команды будет произведен выход из программы|
| **Enter**               | Будет сделан 1 шаг процессора|

---

### Выполнение программы

**Примерный алгоритм выполнения:**
* Инициализация
* Вечный цикл:
  * Ждать действий пользователя
  * Обработать действия пользователя
  * Шаг:
      * Прочитать из памяти код инструкции
      * Получить структуру ИНСТРУКЦИЯ по ее коду
      * Обработать 1 и 2 аргумент
      * Выполнить операцию
      * Увеличить счетчик программы



## Компилятор

### Синтаксис

По скольку мне не очень нравиться синтаксис 8 битных ассемблеров, я решил использовать чуть модифицированный синтаксис Intel

```
jmp start


start:
  .loop:
    cmp ax, 10
    je .finish
    add ax, 1
    jmp .loop
  .finish:
  hlt
```
---
Паттерн инструкции выглядит так

`<имя инструкции> (<тип данных>) <аргумент 1>, <аргумент 2>`

Компилятор старается автоматически определить тип данных, но выходит это не всегда поэтому иногда нужно указывать их вручную (**byte (b)**, **word (w)**)

Например команда `mov b [0x0100], [0x0700]` перенесет **1** байт из памяти по адресу **0x0700** в память по адресу **0x0100**
а команда `mov w [0x0100], [0x0700]` перенесет уже **2** байта из памяти по адресу **0x0700** в память по адресу **0x0100**

---

### Бинарные файлы
После "компиляции" на выходе мы получаем голый бинарник с машинными кодами к эмулятору

Для того чтобы заглянуть в бинарник рекомендую использовать программку [HxD](https://mh-nexus.de/en/hxd/)

Пока что не получиться запустить бинарник на эмуляторе (Он пока что не умеет их читать)

---

### Параметры командной строки

Для компиляции нужно в консоль ввести команду вида

`путь\к\ASMC.exe -i "путь\к\исходнику.asm" -o "путь\к\бинарнику.bin"`

|   Параметр  | Описание |
|      --     |     --      |
| --help, -h  | Получить помощь |
| --in, -i    | Входной файл |
| --out, -o   | Выходной файл |
| --debug, -d | Показывать манипуляции компилятора с кодом (нужно для выявления ошибок) |

---

### Выполнение программы

**Примерный алгоритм работы:**
* Проверка и парсинг параметров командной строки
* Синтаксический анализ:
  * Прочитать файл
  * Пройтись построчно по нему
  * Распарсить строки на лексемы
* Лексический анализ и код генерация:
  * Подсчет размеров будущей программы
  * Поиск и замена именованных указателей в программе
  * Генерация кода
* Запись в файл.bin


## FAQ

* По поводу вопросов добро пожаловать в мой [Телеграм](https://t.me/Vxyxol)  или @Vxyxol

### В планах
1. Переделать архитектуру приложения
2. Усложнить список инструкций (ADD, SUB, PUSH, POP, CALL, RET ...)
3. Добавить стек
4. Добавить постоянную память
5. Добавить графику



#### **Вдохновлено** проектом Дэвида Бара (javidx9) [olcNES Emulator](https://github.com/OneLoneCoder/olcNES)

