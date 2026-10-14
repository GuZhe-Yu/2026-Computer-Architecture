# SAD 測試

測試程式位於上一層的 [`test_sad.c`](../test_sad.c)，會呼叫 [`sad_accumulate`](../sad.c) 並逐筆比對預期結果。

## 執行

```text
cc -std=c11 -Wall -Wextra -Werror ../sad.c ../test_sad.c -o test_sad
./test_sad
```

成功時應輸出：

```text
12 tests passed.
```

測試向量也整理在 [`test_vectors.csv`](test_vectors.csv)。
