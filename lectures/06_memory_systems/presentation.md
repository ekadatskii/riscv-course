---
marp: true
title: "Лекция 6. Память, защита и прерывания в RISC-V-системе"
description: "Финальная лекция курса RISC-V для FPGA"
theme: default
paginate: true
size: 16:9
math: mathjax
style: |
  section {
    font-family: Arial, Helvetica, sans-serif;
    font-size: 26px;
    color: #151515;
    background: #fbfbfd;
  }
  h1 {
    font-size: 44px;
    color: #182033;
  }
  h2 {
    font-size: 36px;
    color: #182033;
  }
  h3 {
    font-size: 28px;
    color: #24324d;
  }
  code {
    font-family: "JetBrains Mono", "Fira Code", Consolas, monospace;
    background: #eef1f8;
    border-radius: 6px;
    padding: 0.05em 0.25em;
  }
  pre code {
    font-size: 21px;
    line-height: 1.35;
  }
  table {
    font-size: 22px;
  }
  .small {
    font-size: 20px;
  }
  .tiny {
    font-size: 17px;
  }
  .center {
    text-align: center;
  }
  .muted {
    color: #697386;
  }
  .accent {
    color: #2854b8;
    font-weight: 700;
  }
  .cols {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 28px;
    align-items: start;
  }
  .cols-3 {
    display: grid;
    grid-template-columns: 1fr 1fr 1fr;
    gap: 22px;
    align-items: start;
  }
  .box {
    border: 2px solid #cfd8ef;
    border-radius: 14px;
    padding: 18px 22px;
    background: #ffffff;
  }
  .blue {
    border-color: #96b7ff;
    background: #eef4ff;
  }
  .green {
    border-color: #9ad7aa;
    background: #effaf2;
  }
  .yellow {
    border-color: #e5ca76;
    background: #fff8df;
  }
  .red {
    border-color: #e89a9a;
    background: #fff0f0;
  }
  .diagram {
    font-family: "JetBrains Mono", "Fira Code", Consolas, monospace;
    font-size: 21px;
    line-height: 1.35;
    background: #ffffff;
    border: 2px solid #d7deee;
    border-radius: 14px;
    padding: 18px 22px;
    white-space: pre;
  }
  .big {
    font-size: 34px;
    line-height: 1.35;
  }
  .footer-note {
    position: absolute;
    bottom: 26px;
    left: 70px;
    right: 70px;
    font-size: 18px;
    color: #697386;
  }
---

# Лекция 6

## Память, защита памяти и прерывания в RISC-V-системе

---

## Содержание, часть 1

1. [От ядра к системе](#3)
2. [Почему память тормозит процессор](#5)
3. [Memory hierarchy](#6)
4. [Cache memory](#8)
5. [Cache line, tag/index/offset](#9)
6. [Hit, miss и политики записи](#12)
7. [Cache в FPGA](#16)
8. [Virtual memory](#19)

---

## Содержание, часть 2

9. [Address translation и page tables](#23)
10. [TLB и page fault](#26)
11. [Memory protection](#29)
12. [PMP](#32)
13. [Exceptions и interrupts](#34)
14. [Trap mechanism в RISC-V](#35)
15. [Interrupt controller](#39)
16. [Итоговая SoC-картина](#44)

---

## 1. В предыдущей лекции

В лекции 5 мы рассмотрели, как процессор исполняет больше инструкций за единицу времени:

- branch prediction;
- speculation;
- out-of-order execution;
- register renaming;
- multithreading и SMT.

<div class="box blue">
Но быстрый конвеер бесполезен, если каждая инструкция ждёт медленную память или процессор не умеет безопасно работать с ОС и периферией.
</div>

---

## Что добавляется на системном уровне

<style scoped>
.grid-2 {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 28px;
  font-size: 23px;
  line-height: 1.18;
}

.grid-2 h3 {
  font-size: 26px;
  margin: 0 0 10px 0;
}

.grid-2 ul {
  margin: 0;
  padding-left: 1.1em;
}

.grid-2 li {
  margin-bottom: 4px;
}

.note {
  margin-top: 18px;
  font-size: 21px;
  line-height: 1.25;
  padding: 10px 14px;
}
</style>

<div class="grid-2">

<div>

### Память

- I-cache и D-cache
- RAM
- memory bus
- virtual memory
- address translation
- page tables
- memory protection

</div>

<div>

### События и периферия

- memory-mapped peripherals
- exceptions
- interrupts
- interrupt controller
- timer interrupt
- external interrupt
- trap handler

</div>

</div>

</div>

---

## RISC-V SoC

<style scoped>
.soc-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 14px;
  font-size: 21px;
}

.soc-box {
  flex: 1;
  min-height: 120px;
  border: 2px solid #d6b46a;
  border-radius: 14px;
  padding: 14px;
  background: rgba(255, 248, 225, 0.65);
  text-align: center;
}

.soc-box strong {
  display: block;
  font-size: 24px;
  margin-bottom: 10px;
}

.soc-arrow {
  font-size: 28px;
}

.note {
  margin-top: 28px;
  font-size: 22px;
  line-height: 1.3;
  padding: 12px 16px;
}
</style>

<div class="soc-row">

<div class="soc-box">
<strong>CPU core</strong>
datapath<br>
control<br>
pipeline
</div>

<div class="soc-arrow">→</div>

<div class="soc-box">
<strong>Cache / MMU</strong>
I-cache<br>
D-cache<br>
translation
</div>

<div class="soc-arrow">→</div>

<div class="soc-box">
<strong>Bus</strong>
AXI / AHB<br>
Wishbone<br>
interconnect
</div>

<div class="soc-arrow">→</div>

<div class="soc-box">
<strong>Memory / IO</strong>
RAM<br>
peripherals<br>
IRQ controller
</div>

</div>

<div class="note">

На системном уровне мы смотрим не только на ядро, но и на всю SoC: как CPU связан с памятью, периферией и внешними событиями.

</div>

---

## 4. Почему память — узкое место

Каждый такт процессору необходимо:

- брать новую инструкцию;
- читать операнды;
- иногда читать или писать данные в память.

Проблема:

<div class="box red">
ALU может быть быстрой, а внешняя память — на порядки медленнее.
</div>

Если каждый <code>lw</code> ждёт DDR, pipeline регулярно простаивает.

---

## 5. Уровни памяти

Чем ближе память к ядру, тем она быстрее, но тем меньше её объём.

| Уровень | Где находится | Задержка |
|---|---|---:|
| Registers | внутри ядра | ~1 такт |
| L1 cache | рядом с pipeline | ~1–4 такта |
| L2 cache | рядом с ядром / кластером | ~8–20 тактов |
| BRAM / SRAM | внутри FPGA / SoC | ~1–3 такта |
| DDR / DRAM | внешняя память | ~50–300+ тактов |
| Flash / SD | хранилище | 1000+ тактов |

<div class="note">

Числа примерные

</div>

---

## 6. Проблемы связанные с удаленностью памяти

Pipeline может выполнять инструкции каждый такт, но доступ к памяти может занимать десятки или сотни тактов.

| Что хочет CPU | Что может произойти |
|---|---|
| Получить следующую инструкцию | instruction fetch ждёт память |
| Выполнить `lw` | данные ещё не пришли |
| Выполнить `sw` | запись ждёт bus / memory controller |
| Продолжить pipeline | стадия MEM блокирует следующие инструкции |

<div class="note">

Быстрый pipeline не спасает, если он постоянно ждёт медленную память.

</div>

---


## 7. Кэш. Локальность данных
<b>Cache</b> — маленькая быстрая память между ядром и основной памятью, которая хранит копии недавно использованных данных или инструкций

Кэш хорошо работает потому, что программы часто обращаются к памяти предсказуемо.

| Вид локальности | Идея | Пример |
|---|---|---|
| Временная локальность | если данные использовались недавно, они могут понадобиться снова | переменная в цикле |
| Пространственная локальность | если использовался один адрес, скоро могут понадобиться соседние адреса | проход по массиву |

<div class="note">

Кэш использует локальность:  
он хранит рядом с ядром то, что программа, скорее всего, скоро снова попросит.

</div>

---

## 8. Пример работы Кэш

<div class="diagram"><pre>
                 address / request
CPU core ─────────────────────────► Cache
                                    │
                                    │ check tag / valid
                                    ▼
                              ┌───────────┐
                              │ hit/miss? │
                              └─────┬─────┘
                                    │
        ┌───────────────────────────┴───────────────────────────┐
        │                                                       │
      HIT                                                     MISS
        │                                                       │
        ▼                                                       ▼
Cache returns data                              Cache requests line from RAM
        │                                                       │
        ▼                                                       ▼
CPU continues                         RAM → cache line → Cache → CPU continues
</pre></div>

---

## 9. Cache line

Данные обычно переносятся не по одному байту и не по одному слову, а блоками.

<div class="box blue">
<b>Cache line</b> — минимальный блок данных, который cache загружает из памяти.
</div>

Пример:

<div class="diagram">Адреса в памяти:
0x1000  0x1004  0x1008  0x100C
  │       │       │       │
  └───────┴───────┴───────┘
        одна cache line</div>

Если программа читает массив подряд, это выгодно.

---

## Разбиение адреса: tag / index / offset

<style scoped>
section {
  font-size: 22px;
}

section h2 {
  font-size: 34px;
}

section ul {
  margin-top: 0;
  margin-bottom: 10px;
}

section li {
  margin-bottom: 2px;
}

section table {
  font-size: 18px;
  line-height: 1.15;
}

.diagram {
  font-size: 16px;
  line-height: 1.05;
  margin-top: 10px;
  margin-bottom: 10px;
}

pre {
  margin: 0;
}

.small-code {
  font-size: 20px;
}
</style>

Допустим:

- адрес 32-битный;
- cache = 4 KiB;
- cache line = 64 байта;
- количество строк cache = cache / cache line

Тогда:

| Поле | Размер | Зачем нужно |
|---|---:|---|
| `offset` | 6 бит | выбрать байт внутри cache line |
| `index` | 6 бит | выбрать строку cache |
| `tag` | 20 бит | проверить, та ли область памяти лежит в строке |

<div class="diagram"><pre>
31                    12 11       6 5        0
┌──────────────────────┬───────────┬──────────┐
│         tag          │   index   │  offset  │
│        20 бит        │   6 бит   │  6 бит   │
└──────────────────────┴───────────┴──────────┘
</pre></div>

---

## 11. Кэш прямого отображения. Direct-mapped cache

В direct-mapped cache каждый адрес может попасть только в одну конкретную строку cache.

<div class="diagram">
index = часть адреса
Адрес A ──► строка 0
Адрес B ──► строка 1
Адрес C ──► строка 2
Адрес D ──► строка 3</div>

<div class="diagram">
Адреса - которые могут вытеснять друг друга
0x8000_1040 
0x8000_2040
0x8000_3040
0x8000_4040</div>

Плюс: простая и быстрая схема.
Минус: разные адреса могут постоянно вытеснять друг друга.

---

## 12. Cache hit и cache miss

<div class="cols">
<div class="box green">
<h3>Cache hit</h3>
Данные уже есть в cache.

CPU получает данные быстро.
</div>
<div class="box red">
<h3>Cache miss</h3>
Данных нет в cache.

Нужно идти в память, загрузить cache line и подождать.
</div>
</div>

<br>

<div class="diagram">CPU request
   │
   ▼
check cache
   │
   ├── hit  ──► return data
   │
   └── miss ──► request RAM ──► fill line ──► return data</div>

---

## 13. Valid bit, dirty bit, tag

Обычно строка cache хранит не только данные.

| Поле | Зачем нужно |
|---|---|
| Data | Сама cache line |
| Tag | Проверка, какие адреса лежат в этой строке |
| Valid bit | Есть ли в строке корректные данные |
| Dirty bit | Были ли данные изменены и ещё не записаны в RAM |

---

## 14. Политики записи

<div class="cols">
<div class="box blue">
<h3>Write-through</h3>
Запись идёт и в cache, и сразу в память.

</div>
<div class="box yellow">
<h3>Write-back</h3>
Запись сначала меняет только cache line.

В память данные уходят позже, при вытеснении.
</div>
</div>

<br>

<div class="box">
В write-back cache нужен <code>dirty bit</code>, чтобы понимать, надо ли сохранять строку обратно в память.
</div>

---

## 15. Write-allocate и no-write-allocate

Что делать при записи, если адреса нет в cache?

| Политика | Идея |
|---|---|
| Write-allocate | Сначала загрузить cache line, потом записать в неё |
| No-write-allocate | Не загружать строку в cache, а писать напрямую дальше |

Обычно:

- write-back часто сочетается с write-allocate;
- write-through может сочетаться с no-write-allocate.

---

## 16. Пример: <code>lw</code> через cache

```asm
lw t0, 0(a0)
```

<div class="diagram">1. ALU считает адрес: a0 + 0
2. D-cache делит адрес на tag/index/offset
3. По index выбирается строка cache
4. Сравнивается tag
5. Если valid=1 и tag совпал → hit
6. Если нет → miss, запрос в RAM
7. После загрузки cache line нужное слово попадает в t0</div>

---

## 17. Cache в FPGA: из чего он состоит

<div class="diagram">┌───────────────────────────────────┐
│              D-cache              │
├───────────────────────────────────┤
│ data RAM: cache lines             │
│ tag RAM: tags + valid + dirty     │
│ comparator: проверка tag          │
│ FSM: miss / refill / write-back   │
└───────────────────────────────────┘</div>

<div class="box blue">
Обычно это BRAM/URAM + компаратор + контроллер состояний.
</div>

---

## 18. I-cache и D-cache

<div class="cols">
<div class="box green">
<h3>I-cache</h3>
Хранит инструкции.

Помогает не тормозить instruction fetch.
</div>
<div class="box blue">
<h3>D-cache</h3>
Хранит данные.

Помогает ускорить <code>lw</code>, <code>sw</code> и работу со стеком.
</div>
</div>

<br>

<div class="diagram">        ┌──────────┐
PC ───► │ I-cache │ ───► instruction
        └──────────┘

ALU addr ─► │ D-cache │ ───► data</div>

---

## 19. Почему доступ к памяти становится сложнее

Без кэша кажется, что каждое обращение CPU сразу идёт в RAM.

С кэшем это уже не так.

| Что делает программа | Что может произойти на самом деле |
|---|---|
| `lw` читает адрес | данные могут прийти из D-cache, а не из RAM |
| `sw` пишет адрес | запись может остаться в cache и попасть в RAM позже |
| CPU читает инструкцию | инструкция может прийти из I-cache |
| устройство пишет в память через DMA | CPU может видеть старую копию в D-cache |


<div class="note">

Кэш ускоряет доступ к памяти, но при этом добавляются некоторые сложности, которые приходится учитывать

</div>

---

## 20. Virtual memory: зачем нужны виртуальные адреса

<style scoped>
.vm-diagram {
  display: grid;
  grid-template-columns: 1fr 0.45fr 1fr;
  gap: 18px;
  align-items: center;
  font-size: 22px;
  line-height: 1.25;
}

.vm-box {
  border: 2px solid #d6b46a;
  border-radius: 14px;
  padding: 14px 16px;
  background: rgba(255, 248, 225, 0.65);
}

.vm-box h3 {
  margin: 0 0 10px 0;
  font-size: 25px;
}

.vm-box ul {
  margin: 0;
  padding-left: 1.1em;
}

.vm-arrow {
  text-align: center;
  font-size: 24px;
  line-height: 1.2;
}

.note {
  margin-top: 22px;
  font-size: 21px;
  line-height: 1.25;
  padding: 10px 14px;
}
</style>

<div class="vm-diagram">

<div class="vm-box">

### Virtual address space

Процесс A видит:

- code
- stack
- heap
- libraries

Процесс B видит:

- code
- stack
- heap
- libraries

</div>

<div class="vm-arrow">

VA  
↓  
MMU + page tables  
↓  
PA

</div>

<div class="vm-box">

### Physical memory

Реальная RAM:

- страницы процесса A
- страницы процесса B
- страницы kernel
- memory-mapped devices

</div>

</div>

<div class="note">

Виртуальные адреса дают каждому процессу своё адресное пространство,  
а MMU переводит эти адреса в реальные физические адреса RAM.

</div>

---

## 21. Physical address vs virtual address

<div class="cols">
<div class="box green">
<h3>Physical address</h3>
Реальный адрес на шине памяти.

Именно его видит RAM и контроллер памяти.
</div>
<div class="box blue">
<h3>Virtual address</h3>
Адрес, который использует программа.

Перед доступом к памяти он должен быть переведён в physical address.
</div>
</div>

<br>

<div class="diagram">virtual address ──► MMU ──► physical address</div>

---

## 22. Зачем нужна виртуальная память

Virtual memory нужна не только ради «большой памяти».

| Возможность | Что даёт |
|---|---|
| Изоляция процессов | Один процесс не портит память другого |
| Защита ОС | User-код не пишет в kernel memory |
| Удобная загрузка программ | Каждая программа может иметь похожую карту памяти |
| Demand paging | Страницы можно подгружать по необходимости |
| Memory-mapped files | Файл может выглядеть как область памяти |

---

## 23. Страницы памяти

Память делится на страницы фиксированного размера.

Типичный размер страницы: **4 KiB**.

<div class="diagram">Virtual address space

┌─────────┐ page 0
├─────────┤ page 1
├─────────┤ page 2
├─────────┤ page 3
└─────────┘ ...</div>

Перевод идёт не для каждого байта отдельно, а для страницы.

---

## 24. Page table

<div class="big">
<b>Page table</b> — таблица, которая говорит, куда виртуальная страница отображается в физической памяти.
</div>

<br>

<div class="diagram">Virtual page 5 ──► Physical page 12
Virtual page 6 ──► Physical page 21
Virtual page 7 ──► not present
Virtual page 8 ──► Physical page 3</div>

Если страницы нет или доступ запрещён, возникает исключение.

---

## 25. Address translation

Виртуальный адрес делится на:

<div class="diagram">┌──────────────────────┬──────────────────┐
│ Virtual Page Number  │   Page Offset    │
└──────────────────────┴──────────────────┘
           │                    │
           │                    └─ остаётся без изменений
           │
           ▼
       page table
           │
           ▼
┌──────────────────────┬──────────────────┐
│ Physical Page Number │   Page Offset    │
└──────────────────────┴──────────────────┘</div>

---

## 27. TLB

Доступ в page table для каждого доступа к памяти - дорогая операция

<div class="box blue">
<b>TLB</b> — Translation Lookaside Buffer. Это cache для трансляций virtual page → physical page.
</div>

<div class="diagram">virtual address
   │
   ▼
check TLB
   │
   ├── TLB hit  ──► physical address
   │
   └── TLB miss ──► page table walk ──► update TLB</div>

---

## 28. Page fault

Page fault возникает, когда адрес нельзя нормально перевести или использовать.

Причины:

- страница не присутствует в памяти;
- нет прав на чтение, запись или исполнение;
- обращение нарушает правила режима доступа;
- таблица страниц содержит некорректную запись.

<div class="box yellow">
Page fault — иногда ОС может обработать его и продолжить выполнение программы.
</div>

---

## 29. Bare-metal, RTOS и Linux

| Система | Обычно нужна MMU? | Комментарий |
|---|---|---|
| Bare-metal | нет | Код сам управляет памятью |
| Маленькая RTOS | редко | Может хватить MPU/PMP |
| Linux | да | Нужна полноценная virtual memory |

MPU/PMP - Memory Protection Unit/Physical Memory Protection

<div class="box blue">
Многие простые FPGA RISC-V cores могут жить без MMU. Но для Linux-class системы MMU становится обязательной частью дизайна.
</div>

---

## 30. Memory protection: зачем запрещать доступ

Без защиты памяти любой код может:

- испортить память ядра ОС;
- записать в память другого процесса;
- выполнить данные как код;
- случайно обратиться к периферии;
- сломать систему из-за ошибки указателя.

<div class="box red">
Memory protection нужна не только от злоумышленника, но и от обычных багов.
</div>

---

## 31. Уровни привилегий в RISC-V

RISC-V разделяет уровни привилегий.

| Режим | Назначение |
|---|---|
| M-mode | Machine mode, самый привилегированный уровень |
| S-mode | Supervisor mode, обычно ядро ОС |
| U-mode | User mode, обычные программы |

---

## 32. Page permissions

Запись page table может содержать права доступа.

| Бит | Смысл |
|---|---|
| R | страницу можно читать |
| W | в страницу можно писать |
| X | из страницы можно исполнять инструкции |
| U | доступна из user mode |
| V | запись валидна |

---

## 33. PMP — Physical Memory Protection

<div class="big">
<b>PMP</b> защищает физические области памяти.
</div>

<br>

Он может работать даже без виртуальной памяти.

<div class="diagram">Physical memory

0x0000_0000 ─┬─ ROM:  R X
             │
0x2000_0000 ─┼─ RAM:  R W X
             │
0x4000_0000 ─┴─ MMIO: R W, no execute</div>

PMP полезен для embedded и bare-metal систем.

---

## 34. MMU vs PMP

| Механизм | Что делает | Где особенно полезен |
|---|---|---|
| MMU | Переводит virtual address в physical address | Linux, процессы, virtual memory |
| Page permissions | Защищают страницы виртуальной памяти | User/Supervisor разделение |
| PMP | Защищает физические регионы памяти | Bare-metal, RTOS, firmware |

<div class="box blue">
MMU отвечает за трансляцию адресов. PMP — за аппаратные границы доступа к physical memory.
</div>

---

## 35. Exceptions и interrupts

| Событие | Когда возникает | Пример |
|---|---|---|
| Exception | Из-за текущей инструкции | illegal instruction, load fault, page fault |
| Interrupt | Асинхронно, извне потока инструкций | timer, UART, GPIO, DMA |

<div class="box yellow">
Оба типа событий в RISC-V попадают в общий механизм — <b>trap</b>.
</div>

---

## 36. Trap mechanism

<div class="big">
<b>Trap</b> — переход процессора в специальный обработчик события.
</div>

<br>

<div class="diagram">normal program
   │
   │ exception / interrupt
   ▼
trap handler
   │
   │ обработка события
   ▼
return to program</div>

Trap позволяет ОС или firmware перехватить ситуацию и принять решение.

---

## 37. Важные CSR для trap в M-mode

| CSR | Зачем нужен |
|---|---|
| <code>mtvec</code> | адрес обработчика trap |
| <code>mepc</code> | адрес инструкции, куда нужно вернуться |
| <code>mcause</code> | причина trap |
| <code>mtval</code> | дополнительная информация: адрес, инструкция и т.п. |
| <code>mstatus</code> | флаги состояния и разрешения прерываний |

<div class="footer-note">
Для S-mode существуют аналогичные supervisor CSR: <code>stvec</code>, <code>sepc</code>, <code>scause</code>, <code>stval</code>, <code>sstatus</code>.
</div>

---

## 38. Вход в обработчик

При trap процессор:

1. определяет причину события;
2. записывает причину в <code>mcause</code>;
3. сохраняет адрес возврата в <code>mepc</code>;
4. обновляет состояние в <code>mstatus</code>;
5. переходит по адресу из <code>mtvec</code>.

<div class="diagram">event ──► save cause/pc/status ──► jump to mtvec</div>

---

## 39. Возврат через <code>mret</code>

Когда обработчик закончил работу, нужно вернуться назад.

```asm
mret
```

<code>mret</code>:

- возвращаемся в исходное состояние;
- восстанавливаем состояние interrupt enable;
- возвращает PC из <code>mepc</code>.

---

## 40. Зачем нужен interrupt controller

У системы может быть много источников прерываний:

- UART;
- GPIO;
- SPI/I2C;
- Ethernet;
- DMA;
- таймеры;
- внешние IP-блоки в FPGA.

<div class="box blue">
Interrupt controller собирает запросы от устройств и помогает CPU понять, какое событие нужно обработать первым.
</div>

---

## 41. Interrupt controller как диспетчер

<div class="diagram">UART ─────┐
GPIO ─────┤
DMA  ─────┤
Timer ────┤
Custom IP ┤
          ▼
 ┌────────────────────┐
 │ interrupt controller│
 └─────────┬──────────┘
           │ IRQ
           ▼
        RISC-V CPU</div>

Контроллер может хранить pending-биты, enable-биты, приоритеты и номер активного IRQ.

---

## 42. Pending / enable / priority

| Механизм | Смысл |
|---|---|
| Pending | событие уже произошло и ждёт обработки |
| Enable | разрешено ли это прерывание |
| Priority | насколько оно важное относительно других |
| Claim | CPU забирает номер IRQ в обработку |
| Complete | CPU сообщает, что обработка завершена |


---

## 43. local interrupts и external interrupts

В RISC-V-платформах часто разделяют:

<div class="cols">
<div class="box green">
<h3>Local interrupts</h3>
Софтовые прерывания и прерывания таймера, близкие к ядру.

</div>
<div class="box blue">
<h3>External interrupts</h3>
Прерывания от внешней периферии.

</div>
</div>

<br>

---

## 44. Пример: UART interrupt

<div class="diagram">1. UART получил байт
2. UART выставил interrupt request
3. Interrupt controller пометил IRQ как pending
4. CPU получил external interrupt
5. CPU вошёл в trap handler по mtvec
6. Handler прочитал причину события
7. Handler забрал байт из UART RX-регистра
8. Handler сообщил interrupt controller: complete
9. CPU вернулся через mret</div>

---


## 45. Итоговая SoC-картина

<style scoped>
section {
  font-size: 24px;
}

section h2 {
  font-size: 32px;
  margin-bottom: 10px;
}

.diagram {
  font-size: 19px;
  line-height: 1.0;
  margin-top: 6px;
}

.diagram pre {
  margin: 0;
}
</style>

<div class="diagram"><pre>
                 ┌────────────────────┐
                 │    RISC-V core     │
                 │ pipeline/CSR/PMP   │
                 └─────────┬──────────┘
                           │
              ┌────────────┴────────────┐
              │                         │
         ┌────▼────┐               ┌────▼────┐
         │ I-cache │               │ D-cache │
         └────┬────┘               └────┬────┘
              │                         │
              └────────────┬────────────┘
                           │
                     ┌─────▼─────┐
                     │ MMU / TLB │
                     └─────┬─────┘
                           │
                     ┌─────▼─────┐
                     │    Bus    │
                     └─────┬─────┘
      ┌────────────────────┼────────────────────┐
      │                    │                    │
  ┌───▼───┐           ┌────▼────┐          ┌────▼────┐
  │  RAM  │           │  UART   │          │  Timer  │
  └───────┘           └────┬────┘          └────┬────┘
                           │                    │
                           └────────┬───────────┘
                                    ▼
                                IRQ ctrl
</pre></div>

---

## 47. Что мы прошли за курс

| Уровень | Что изучали |
|---|---|
| ISA | инструкции, регистры, форматы команд |
| Datapath | ALU, registers, memory, immediate generation |
| Control | сигналы управления, выполнение инструкции |
| Pipeline | стадии, hazards, forwarding, stalls |
| Performance | prediction, speculation, OoO, multithreading |
| System | cache, MMU, protection, traps, interrupts |

---

## 48.Что еще можно изучить и сделать

- написать простой direct-mapped cache;
- добавить memory-mapped UART и timer;
- сделать trap handler на bare-metal C/ASM;
- попробовать PMP-регионы;
- изучить AXI/AHB/Wishbone interconnect;
- посмотреть open-source RISC-V cores;
- отдельно изучить Linux-capable RISC-V SoC.


