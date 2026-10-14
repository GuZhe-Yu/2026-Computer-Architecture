# SAD：Sum of Absolute Differences

## 題目資訊

| 項目 | 內容 |
|---|---|
| 指令名稱 | `SAD` |
| 題目類別 | AI / 視覺類 |
| 運算元類型 | 二元（累加） |
| 輸入定義 | `rd` 為累加器，同時是來源與目的 |
| 語意公式 | `rd_new = rd_old + \|signed(rs1) - signed(rs2)\|` |
| 溢位處理 | 結果保留低 32 bits（modulo `2^32`） |

SAD 用來計算兩個 signed 32-bit 數值的絕對差，並加到 `rd` 原本的累加值。執行時必須先讀取 `rd_old`、`rs1` 與 `rs2`，完成計算後再將結果寫回 `rd`。

正式語意依課程規格：[HOMEWORK.md 第六節](../../../HOMEWORK.md#六題目語意規格)。

## 實作位置

| 內容 | 位置 |
|---|---|
| CP1 C 參考實作與測試 | [`../../src/c_baseline/`](../../src/c_baseline/) |
| CP2 RV32IM baseline 與指令統計 | [`../../src/rv32im/`](../../src/rv32im/) |
| CP3 指令規格與編碼 | 本檔「指令規格」 |
| 報告 | [`../../report/`](../../report/) |
| 個人貢獻 | [`../../CONTRIBUTIONS.md`](../../CONTRIBUTIONS.md) |

## 指令規格

### 語意（公式）

```text
rd_new = rd_old + |signed(rs1) - signed(rs2)|
```

`rd_old` 是執行指令前的 `rd` 值。硬體先讀取 `rd_old`、`rs1`、`rs2`，計算完成後才將結果寫回 `rd`。

### 運算元與定義域

- `rs1`、`rs2`：signed 32-bit 二補數整數。
- `rd`：32-bit 累加器，同時是來源與目的。
- 三個暫存器欄位各為 5 bits。

### 結果範圍與溢位處理

絕對差的數學值範圍是 `0` 至 `0xFFFFFFFF`。累加結果保留低 32 bits：

```text
rd_new = (rd_old + abs(signed(rs1) - signed(rs2))) mod 2^32
```

例如 `rs1 = INT32_MAX`、`rs2 = INT32_MIN` 時，絕對差是 `0xFFFFFFFF`。

### 虛擬碼

```text
function SAD(rd_old, rs1, rs2):
    if signed(rs1) >= signed(rs2):
        difference = unsigned(rs1) - unsigned(rs2)
    else:
        difference = unsigned(rs2) - unsigned(rs1)
    return (rd_old + difference) mod 2^32
```

使用 unsigned 減法表示絕對差，避免 `INT32_MIN` 取負值或 signed overflow。

### R-type 編碼

```text
31        25 24    20 19    15 14  12 11     7 6       0
+------------+--------+--------+------+--------+---------+
|   funct7   |  rs2   |  rs1   |funct3|   rd   | opcode  |
+------------+--------+--------+------+--------+---------+
```

- `opcode = 0001011`（`0x0B`，custom-0）。
- `funct7` 與 `funct3` 需使用教師分配值；目前題目資料未提供固定值，因此不能自行指定正式編碼。
- 欄位組合公式：

```text
encoding = (funct7 << 25) | (rs2 << 20) | (rs1 << 15)
         | (funct3 << 12) | (rd << 7) | 0x0B
```

欄位範例（假設值僅供示範，不代表教師正式分配）：

```text
SAD x5, x6, x7
funct7=0000000, rs2=00111, rs1=00110, funct3=000, rd=00101
machine code = 0x0073028B
```

### 測試向量

| `rd_old` | `rs1` | `rs2` | `rd_new` |
|---:|---:|---:|---:|
| 100 | 10 | 13 | 103 |
| 0 | -10 | 13 | 23 |
| 77 | 42 | 42 | 77 |
| 0 | `INT32_MAX` | `INT32_MIN` | `0xFFFFFFFF` |

完整的 12 組測資見 [`../../src/c_baseline/test_vectors.csv`](../../src/c_baseline/test_vectors.csv)。

### RV32IM baseline 對照

對應片段見 [`../../src/rv32im/README.md`](../../src/rv32im/README.md)：

| 實作 | 靜態指令數 | 動態指令數 |
|---|---:|---:|
| RV32IM baseline | 6 | 5 或 6 |
| SAD 自訂指令 | 1 | 1 |

baseline 使用 `slt`、`sub`、`beq`、`add` 完成比較、絕對差與累加。

## 參考資料

僅記錄連結與使用目的，不複製參考來源的完整內容。

| 編號 | 名稱 | 來源 | 用途 | 存取日期 |
|---|---|---|---|---|
| R1 | RISC-V Unprivileged ISA | [RISC-V Instruction Set Manual, Volume I, RV32I](https://docs.riscv.org/reference/isa/unpriv/rv32.html)，Custom encodings 與 R-type instruction format | 確認 RV32I 暫存器、R-type 欄位與 custom-0 opcode 的使用方式 | 2026-10-13 |
| R2 | Sum of absolute differences | [Wikipedia: Sum of absolute differences](https://en.wikipedia.org/wiki/Sum_of_absolute_differences)，Definition | 確認逐元素絕對差再累加的演算法定義 | 2026-10-13 |
