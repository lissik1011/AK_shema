# Арифметические инструкции ALU (add, sub, and, ori, slli, mv...)

## Инструкции регистр-регистр (add, sub) и регистр-константа (addi, ori)

- IF (Instruction Fetch):
  - PC -> Instruction_Memory.addr
  - M_instr[PC] -> Instruction Register (IR)
  - PC + 4 -> PC

- ID (Instruction Decode):
  - IR[19:15] -> RegFile.rs1_addr => Вывод rs1_val
  - IR[24:20] -> RegFile.rs2_addr => Вывод rs2_val (только для R-type: add, sub)
  - IR[31:20] -> ImmGen => Вывод imm (только для I-type: addi, ori)
  - Декодирование Control Unit

- EX (Execute):
  - ALU_A = rs1_val
  - ALU_B = rs2_val ИЛИ imm (выбирается через MUX)
  - ALU_result = ALU_A <operation> ALU_B

- MEM (Memory):
  - Память данных не используется. ALU_result просто передается дальше.

- WB (Write Back):
  - ALU_result -> RegFile.wb_data
  - IR[11:7] -> RegFile.rd_addr
  - RegWrite = 1 (Запись в регистр rd)

---

# Инструкции работы с памятью

## Load (lw, lb)

- IF: Чтение инструкции по PC, PC = PC + 4
- ID: Чтение rs1_val из RegFile, генерация смещения imm через ImmGen.
- EX: ALU складывает базовый адрес и смещение: ALU_result = rs1_val + imm.
- MEM (Memory Access):
  - ALU_result -> Data_Memory.addr
  - MemRead = 1
  - Data_Memory[addr] -> mem_data_out
- WB: 
  - mem_data_out -> RegFile.wb_data (через MUX)
  - Запись в rd_addr.

## Store (sw, sb)

- IF: Чтение инструкции по PC, PC = PC + 4
- ID: Чтение rs1_val (база) и rs2_val (данные для записи) из RegFile. Генерация imm.
- EX: ALU_result = rs1_val + imm.
- MEM:
  - ALU_result -> Data_Memory.addr
  - rs2_val -> Data_Memory.write_data
  - MemWrite = 1 (Запись в память)
- WB: 
  - Ничего не происходит (RegWrite = 0).

---

# Команды перехода

## Branch (beq, bne, bgt)

- IF: Чтение инструкции по PC, PC = PC + 4
- ID: Чтение rs1_val и rs2_val из RegFile. Генерация imm (смещение перехода).
- EX: 
  - Branch Target = PC_старый + imm
  - ALU или Компаратор сравнивает rs1_val и rs2_val (например, вычитанием).
  - Если условие истинно, генерируется сигнал branch_taken = 1.
- MEM: 
  - Если branch_taken == 1, то Branch Target -> PC (перезапись адреса следующей инструкции).
- WB: Ничего не происходит.

## Jump and Link (jal)

- IF: Чтение инструкции по PC, PC = PC + 4
- ID: Генерация imm (смещение).
- EX: Branch Target = PC_старый + imm.
- MEM: Branch Target -> PC.
- WB: PC_старый + 4 -> RegFile.wb_data (сохранение адреса возврата в rd, обычно ra).

## Jump Register (jr)

- IF: Чтение инструкции по PC, PC = PC + 4
- ID: Чтение rs1_val из RegFile.
- EX: ALU_result = rs1_val (проброс значения).
- MEM: ALU_result -> PC.
- WB: Ничего не происходит.
