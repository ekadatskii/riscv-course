---
marp: true
paginate: true
---

## 1. Уровни абстракции в вычислительной системе

<!-- DIAG OPEN-->
<div style="display:flex; gap:28px; align-items:flex-start;">

<!-- LEFT OPEN-->
<div style="width:42%; display:flex; flex-direction:column; gap:6px;">

<div style="background:#2F80ED; color:#fff; padding:10px 10px; border-radius:10px; font-weight:700; text-align:center; font-size:18px; line-height:1.1;">
    Код на языке высокого уровня
</div>

<div style="display:flex; align-items:center; gap:10px; line-height:1;">
    <div style="width:28px; text-align:center; font-size:22px; font-weight:800; color:#555;">↓</div>
    <div style="font-size:16px; font-weight:700; color:#333;">Компилятор</div>
</div>

<div style="background:#27AE60; color:#fff; padding:10px 10px; border-radius:10px; font-weight:700; text-align:center; font-size:18px; line-height:1.1;">
    Программа на языке ассемблера
</div>

<div style="display:flex; align-items:center; gap:10px; line-height:1;">
    <div style="width:28px; text-align:center; font-size:22px; font-weight:800; color:#555;">↓</div>
    <div style="font-size:16px; font-weight:700; color:#333;">Ассемблер</div>
</div>

<div style="background:#F2994A; color:#fff; padding:10px 10px; border-radius:10px; font-weight:700; text-align:center; font-size:18px; line-height:1.1;">
    Машинный код
</div>

<div style="display:flex; align-items:center; gap:10px; line-height:1;">
    <div style="width:28px; text-align:center; font-size:22px; font-weight:800; color:#555;">↓</div>
    <div style="font-size:16px; font-weight:700; color:#333;">Выполнение инструкций процессором</div>
</div>

<div style="background:#9B51E0; color:#fff; padding:10px 10px; border-radius:10px; font-weight:700; text-align:center; font-size:18px; line-height:1.1;">
    Описание аппаратной архитектуры
</div>

<div style="display:flex; align-items:center; gap:10px; line-height:1;">
    <div style="width:28px; text-align:center; font-size:22px; font-weight:800; color:#555;">↓</div>
    <div style="font-size:16px; font-weight:700; color:#333;">Микроархитектура</div>
</div>

<div style="background:#EB5757; color:#fff; padding:10px 10px; border-radius:10px; font-weight:700; text-align:center; font-size:18px; line-height:1.1;">
    Описание логической схемы
</div>

</div>
<!-- LEFT CLOSE-->

<!-- RIGHT OPEN -->
<div style="width:58%; padding-left:10px;">

<!-- margin-top сдвигает код вниз, чтобы он был "напротив" синего блока,
        и "подальше" от него по вертикали -->
<div style="margin-top:-24px; background:#F7F7F7; border:1px solid #DDD; border-radius:12px; padding:10px 12px;">
    <pre style="margin:0; font-size:16px; line-height:1.2;"><code>int add(int a, int b) {
    return a + b;
}</code></pre>
</div>

<div style="margin-top:12px; background:#F7F7F7; border:1px solid #DDD; border-radius:12px; padding:10px 12px;">
  <pre style="margin:0; font-size:16px; line-height:1.2;"><code>0x00B50533  # add a0, a0, a1
0x00008067  # ret (jalr x0, 0(ra))</code></pre>
</div>

<!-- Architecture image -->
<div style="margin-top:12px; background:#F7F7F7; border:1px solid #DDD; border-radius:12px; padding:10px 12px;">
<img src="./assets/slide1_pic1.png"
        style="max-width:100%; max-height:160px; object-fit:contain; border-radius:10px; border:1px solid #EEE;" />
</div>

<!-- Logic circuit image -->
<div style="margin-top:12px; background:#F7F7F7; border:1px solid #DDD; border-radius:12px; padding:10px 12px;">
<img src="./assets/slide1_pic2.png"
        style="max-width:100%; max-height:160px; object-fit:contain; border-radius:10px; border:1px solid #EEE;" />
</div>

</div>
<!-- RIGHT CLOSE-->

</div>
<!-- DIAG CLOSE-->

---

## 2. Модель процессора RISC-V

![](./assets/slide2_pic1.svg)

---

## 3. Модель процессора RISC-V с флагами

![](./assets/slide3_pic1.png)

---

## 4. Базовый набор инструкций RISC-V
![](./assets/slide4_pic1.png)

---

## 4. Базовый набор инструкций RISC-V(2)
![](./assets/slide4_pic2.png)

---

## 5. Кодирование инструкций RISC-V

![](./assets/slide5_pic1.png)

---

## 6. Формат инструкции R

`[31:25] funct7 | [24:20] rs2 | [19:15] rs1 | [14:12] funct3 | [11:7] rd | [6:0] opcode`

| Поле | Биты | Назначение |
|---|---:|---|
| opcode | [6:0] | класс OP (`0110011`) |
| rd | [11:7] | куда записать результат |
| rs1 | [19:15] | 1-й операнд (регистр) |
| rs2 | [24:20] | 2-й операнд (регистр) |
| funct3 | [14:12] | подтип операции |
| funct7 | [31:25] | уточнение (например add/sub, srl/sra) |

---

## 7. Формат инструкции I

`[31:20] imm[11:0] | [19:15] rs1 | [14:12] funct3 | [11:7] rd | [6:0] opcode`

| Поле | Биты | Назначение |
|---|---:|---|
| opcode | [6:0] | класс (IMM / LOAD / JALR / SYSTEM) |
| rd | [11:7] | куда записать результат |
| rs1 | [19:15] | операнд / базовый адрес |
| funct3 | [14:12] | подтип операции |
| imm[11:0] | [31:20] | константа/смещение (sign-extend) |

---

## 8. Формат инструкции S

`[31:25] imm[11:5] | [24:20] rs2 | [19:15] rs1 | [14:12] funct3 | [11:7] imm[4:0] | [6:0] opcode`

| Поле | Биты | Назначение |
|---|---:|---|
| opcode | [6:0] | класс STORE (`0100011`) |
| rs1 | [19:15] | **base**: откуда берём адрес |
| rs2 | [24:20] | **data**: что записываем в память |
| funct3 | [14:12] | размер записи: `sb/sh/sw` |
| imm[11:0] | [31:25] + [11:7] | смещение (sign-extend) |

---

## 9. Формат инструкции B

`[31] imm[12] | [30:25] imm[10:5] | [24:20] rs2 | [19:15] rs1 | [14:12] funct3 | [11:8] imm[4:1] | [7] imm[11] | [6:0] opcode`

| Поле | Биты | Назначение |
|---|---:|---|
| opcode | [6:0] | класс BRANCH (`1100011`) |
| rs1 | [19:15] | 1-й операнд сравнения |
| rs2 | [24:20] | 2-й операнд сравнения |
| funct3 | [14:12] | тип условия (beq/bne/blt/...) |
| imm(B) | разрезан | смещение для `PC` (sign-extend) |

---

## 10. Формат инструкции U

`[31:12] imm[31:12] | [11:7] rd | [6:0] opcode`

| Поле | Биты | Назначение |
|---|---:|---|
| opcode | [6:0] | класс U-type: LUI=`0110111`, AUIPC=`0010111` |
| rd | [11:7] | куда записать результат |
| imm[31:12] | [31:12] | 20 бит immediate (верхняя часть) |

---

## 11. Формат инструкции J

`[31] imm[20] | [30:21] imm[10:1] | [20] imm[11] | [19:12] imm[19:12] | [11:7] rd | [6:0] opcode`
| Поле | Биты | Назначение |
|---|---:|---|
| opcode | [6:0] | JAL (`1101111`) |
| rd | [11:7] | куда записать `PC+4` (обычно `ra`) |
| imm(J) | разрезан | смещение от `PC` (sign-extend) |

---

## 12. Соответствие классу инструкций - блокам

<div style="font-size:20px;">

| Класс (RV32I) | Формат | Какие блоки используются | Что происходит |
|---|---|---|---|
| **OP** (`add/sub/...`) | R | RegFile → ALU → WB → RegFile | rd = f(rs1, rs2) |
| **OP-IMM** (`addi/...`) | I | RegFile → ALU(imm) → WB → RegFile | rd = f(rs1, imm) |
| **LOAD** (`lw/lb/...`) | I | RegFile → ALU(addr) → DataMem(read) → WB → RegFile | rd = Mem[rs1+imm] |
| **STORE** (`sw/sb/...`) | S | RegFile → ALU(addr) → DataMem(write) | Mem[rs1+imm] = rs2 |
| **BRANCH** (`beq/...`) | B | RegFile → ALU(compare) → BranchUnit → PC MUX | PC = PC+imm или PC+4 |
| **JAL** (`jal`) | J | PC+4 → WB → RegFile, PC+imm → PC MUX | rd=PC+4; PC=PC+imm |
| **JALR** (`jalr`) | I | RegFile(rs1)+imm → PC MUX, PC+4 → WB | rd=PC+4; PC=(rs1+imm)&~1 |
| **U-type** (`lui/auipc`) | U | ImmGen/PC → WB → RegFile | rd = imm<<12 или PC+(imm<<12) |

---

## 13. Сигналы управления
<div style="font-size:20px;">

| Сигнал | Куда идёт | Что делает |
|---|---|---|
| **RegWrite** | RegFile | Разрешает запись в `rd` |
| **ALUSrc** | MUX перед входом B ALU | `0`: берём `rs2`, `1`: берём `imm` |
| **ALUOp** | ALU control | Выбирает операцию ALU (add/sub/and/...) по `funct3/funct7` |
| **MemRead** | Data Memory | Чтение из памяти (load) |
| **MemWrite** | Data Memory | Запись в память (store) |
| **MemToReg** | MUX writeback | `0`: в `rd` идёт ALU result, `1`: данные из памяти |
| **Branch** | Branch Unit / PC MUX | Разрешает ветвление (используем результат сравнения) |
| **Jump** | PC MUX | Безусловный переход (`jal/jalr`) |
| **PCSel** *(или PCSrc)* | PC MUX | Выбор следующего PC: `PC+4` / target |
| **ImmSel** | ImmGen/SignExt | Выбирает тип immediate: I/S/B/U/J |

---

## 14. opcode / funct3 / funct7 — три уровня кодирования

| Уровень | Сколько бит | За что отвечает | Пример |
|---|---:|---|---|
| **opcode** | 7 | **Класс** инструкции (формат/тип): load/store/branch/op/jump | `0110011` → OP (R-type) |
| **funct3** | 3 | **Подтип** внутри класса | BRANCH: `000`=beq, `001`=bne |
| **funct7** | 7 | **Уточнение**, когда funct3 недостаточно | OP: `add` vs `sub`, `srl` vs `sra` |

Декодер сначала смотрит `opcode` (что это за класс), затем `funct3`, и при необходимости `funct7`.

---

## 15. Immediate в инструкциях I/S/B/U/J

<div style="font-size:24px;">

| Формат | Где используется | Размер imm (эфф.) | Что означает |
|---|---|---:|---|
| **I-type** | `addi`, `lw`, `jalr` | 12 бит | константа / смещение / base+offset |
| **S-type** | `sw`, `sb`, `sh` | 12 бит | смещение для store (base+offset) |
| **B-type** | `beq`, `blt`, … | 13 бит | PC-relative смещение для ветвления (**кратно 2**) |
| **U-type** | `lui`, `auipc` | 20 бит | верхние 20 бит: `imm << 12` |
| **J-type** | `jal` | 21 бит | PC-relative смещение прыжка (**кратно 2**) |

imm иногда “разрезан” для того чтобы сохранить удобные позиции `rs1/rs2/funct3` в 32-битной инструкции


