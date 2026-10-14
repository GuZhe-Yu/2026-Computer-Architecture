# SAD C baseline

本資料夾提供 SAD 自訂指令的 CP1 C 參考實作、可執行測試與測資。

| 檔案 | 內容 |
|---|---|
| [`sad.h`](sad.h)、[`sad.c`](sad.c) | 參考函式 `sad_accumulate` |
| [`test_sad.c`](test_sad.c) | 12 組測試，逐筆比對預期結果 |
| [`test_vectors.csv`](test_vectors.csv) | 測資與預期輸出 |

## 語意

```text
rd_new = rd_old + |signed(rs1) - signed(rs2)|
```

- `rd_old` 是執行前的累加器值，`rd_new` 是寫回同一個 `rd` 的值。
- `rs1` 與 `rs2` 解讀為 signed 32-bit 二補數。
- 累加結果保留低 32 bits（wrap-around）。
- 先讀取 `rd_old`、`rs1`、`rs2`，完成計算後才寫回 `rd`。

## 測試涵蓋

- 一般絕對差與反向運算元
- 負數與正數
- `rs1 == rs2` 的零差值
- `INT32_MIN`、`INT32_MAX` 邊界
- 最大 signed 絕對差
- 32-bit 累加 wrap-around

## 編譯與執行

在本資料夾執行：

```text
cc -std=c11 -Wall -Wextra -Werror sad.c test_sad.c -o test_sad
./test_sad
```

成功時輸出：

```text
12 tests passed.
```
