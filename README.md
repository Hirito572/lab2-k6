# Лаборатори №2 — k6 Performance Testing

## Оюутны мэдээлэл

* **Оюутны нэр:** Э. Мөнх-Очир
* **Оюутны код:** B232270022
* **Хэрэгсэл:** Grafana k6
* **k6 хувилбар:** v2.2.0
* **Үйлдлийн систем:** macOS (darwin/arm64)
* **Туршилтын target:** https://test.k6.io

---

## 1. Лабораторийн ажлын зорилго

Энэхүү лабораторийн ажлын зорилго нь **Grafana k6** хэрэгслийг ашиглан веб системийн гүйцэтгэлийг ачааллын туршилтаар хэмжих, latency, throughput болон error rate зэрэг үндсэн гүйцэтгэлийн хэмжүүрүүдийг судлахад оршино.

Лабораторийн хүрээнд дараах туршилтуудыг хийсэн:

* Baseline test
* Stages test: 5 → 30 → 100 → 0 VU
* Threshold PASS test
* Threshold FAIL test
* 5 VU / 1 минут
* 30 VU / 1 минут
* 100 VU / 1 минут

Мөн k6-ийн threshold механизмыг ашиглан SLO шаардлага хангагдсан эсэхийг шалгасан.

---

# 2. Туршилтын орчин

Туршилтыг macOS үйлдлийн системтэй компьютер дээр Grafana k6 ашиглан гүйцэтгэсэн.

### k6 хувилбар

```text
k6 v2.2.0 (commit/devel, go1.26.5, darwin/arm64)
```

### Туршилтын target

```text
https://test.k6.io
```

---

# 3. Үндсэн k6 script

Үндсэн load test-д `script.js` файлыг ашигласан.

Script нь HTTP GET request илгээж, response status `200` эсэхийг шалгаад iteration бүрийн дараа 1 секундын `sleep` хийдэг.

Үндсэн threshold:

```text
p(95) < 406.08 ms
error rate < 1%
```

---

# 4. Baseline туршилт

Baseline туршилтыг системийн үндсэн latency-г тодорхойлох зорилгоор бага ачааллын нөхцөлд ажиллуулсан.

### Команд

```bash
k6 run script.js 2>&1 | tee results/run-baseline.txt
```

### Үр дүн

| Үзүүлэлт      |        Үр дүн |
| ------------- | ------------: |
| VUs           |             1 |
| Iterations    |             1 |
| HTTP requests |             2 |
| p(90)         |     275.59 ms |
| p(95)         | **282.41 ms** |
| Average       |     220.99 ms |
| Minimum       |     152.74 ms |
| Maximum       |     289.24 ms |
| Throughput    |    0.82 req/s |
| Error rate    |     **0.00%** |
| Threshold     |      **PASS** |

Шинэ baseline туршилтаар p(95) latency **282.41 ms** гарсан.

Туршилтын бүрэн output:

```text
results/run-baseline.txt
```

---

# 5. SLO / Threshold

Лабораторийн үндсэн SLO threshold:

```text
p(95) < 406.08 ms
error rate < 1%
```

406.08 ms threshold нь лабораторийн өмнөх baseline хэмжилт болох **270.72 ms** дээр үндэслэн:

```text
270.72 × 1.5 = 406.08 ms
```

гэж тооцоолсон утга юм.

Иймээс энэхүү эцсийн туршилтуудад:

* HTTP request-ийн p(95) latency **406.08 ms-ээс бага**
* HTTP error rate **1%-иас бага**

байхыг SLO гэж үзсэн.

> Шинэ baseline-ийн p(95) нь 282.41 ms гарсан боловч лабораторийн туршилтуудад ашигласан SLO threshold нь өмнө тогтоосон **406.08 ms** хэвээр хадгалагдсан.

---

# 6. Stages туршилт

Stages тестээр ачааллыг дараах байдлаар нэмэгдүүлж, дараа нь бууруулсан.

```text
5 VU → 30 VU → 100 VU → 0 VU
```

Нийт хугацаа:

```text
1 минут
```

### Команд

```bash
k6 run stages.js 2>&1 | tee results/run-stages.txt
```

### Үр дүн

| Үзүүлэлт      |          Үр дүн |
| ------------- | --------------: |
| Maximum VU    |             100 |
| Duration      |           1 min |
| p(90)         |       313.77 ms |
| p(95)         |   **335.16 ms** |
| Average       |       212.67 ms |
| Minimum       |        55.96 ms |
| Maximum       |          1.17 s |
| Throughput    | **45.93 req/s** |
| HTTP requests |            2814 |
| Iterations    |            1407 |
| Error rate    |       **0.00%** |
| Threshold     |        **PASS** |

Stages туршилтын үед maximum VU 100 хүрсэн бөгөөд p(95) latency 335.16 ms байсан.

406.08 ms-ийн SLO threshold-ийг хангаж, error rate 0.00% гарсан.

---

# 7. Threshold PASS туршилт

Threshold PASS тестийг 30 VU ачаалалтайгаар 1 минут ажиллуулсан.

### Threshold

```text
p(95) < 406.08 ms
error rate < 1%
```

### Команд

```bash
k6 run threshold-pass.js 2>&1 | tee results/run-threshold-pass.txt
```

### Үр дүн

| Үзүүлэлт      |          Үр дүн |
| ------------- | --------------: |
| VUs           |              30 |
| Duration      |           1 min |
| p(90)         |       361.98 ms |
| p(95)         |   **402.98 ms** |
| Average       |       238.85 ms |
| Minimum       |        56.49 ms |
| Maximum       |          1.92 s |
| Throughput    | **39.12 req/s** |
| HTTP requests |            2424 |
| Iterations    |            1212 |
| Error rate    |       **0.00%** |
| Threshold     |        **PASS** |

p(95) = **402.98 ms** бөгөөд 406.08 ms-ийн threshold-ээс бага тул PASS болсон.

Энэ туршилт нь threshold-д маш ойр үр дүн үзүүлсэн боловч:

```text
402.98 ms < 406.08 ms
```

тул threshold хангагдсан.

---

# 8. Threshold FAIL туршилт

Threshold-ийн FAIL ажиллагааг шалгахын тулд зориудаар маш бага threshold ашигласан.

```text
p(95) < 50 ms
```

### Команд

```bash
k6 run threshold-fail.js 2>&1 | tee results/run-threshold-fail.txt
```

### Үр дүн

| Үзүүлэлт      |          Үр дүн |
| ------------- | --------------: |
| VUs           |              30 |
| Duration      |           1 min |
| p(90)         |       354.86 ms |
| p(95)         |   **384.88 ms** |
| Average       |       228.34 ms |
| Minimum       |        56.86 ms |
| Maximum       |          2.44 s |
| Throughput    | **39.99 req/s** |
| HTTP requests |            2458 |
| Iterations    |            1229 |
| Error rate    |       **0.00%** |
| `p(95)<50ms`  |        **FAIL** |

Бодит p(95):

```text
384.88 ms
```

Шаардлагатай threshold:

```text
< 50 ms
```

Тиймээс:

```text
384.88 ms > 50 ms
```

учраас threshold зориудаар FAIL болсон.

Энэ нь тестийн алдаа биш бөгөөд **intentional FAIL test** юм. Мөн k6 output-ийн төгсгөлд threshold crossed гэсэн алдаа гарсан нь энэ FAIL нөхцөлтэй холбоотой.

---

# 9. 5 VU / 1 минутын load test

5 Virtual User ашиглан 1 минутын турш load test хийсэн.

### Команд

```bash
k6 run --vus 5 --duration 1m script.js 2>&1 | tee results/run-05vu.txt
```

### Үр дүн

| Үзүүлэлт      |         Үр дүн |
| ------------- | -------------: |
| VUs           |              5 |
| Duration      |          1 min |
| p(90)         |      323.37 ms |
| p(95)         |  **349.72 ms** |
| Average       |      231.89 ms |
| Minimum       |       58.10 ms |
| Maximum       |         1.32 s |
| Throughput    | **6.67 req/s** |
| HTTP requests |            404 |
| Iterations    |            202 |
| Error rate    |      **0.00%** |
| Threshold     |       **PASS** |

p(95) = 349.72 ms бөгөөд 406.08 ms threshold-ээс бага байна.

---

# 10. 30 VU / 1 минутын load test

30 Virtual User ашиглан 1 минутын турш load test хийсэн.

### Команд

```bash
k6 run --vus 30 --duration 1m script.js 2>&1 | tee results/run-30vu.txt
```

### Үр дүн

| Үзүүлэлт      |          Үр дүн |
| ------------- | --------------: |
| VUs           |              30 |
| Duration      |           1 min |
| p(90)         |       234.08 ms |
| p(95)         |   **239.09 ms** |
| Average       |       146.61 ms |
| Minimum       |        54.68 ms |
| Maximum       |       456.60 ms |
| Throughput    | **45.41 req/s** |
| HTTP requests |            2760 |
| Iterations    |            1380 |
| Error rate    |       **0.00%** |
| Threshold     |        **PASS** |

30 VU-ийн үед p(95) latency 239.09 ms байсан бөгөөд SLO threshold-ийг хангаж PASS болсон.

---

# 11. 100 VU / 1 минутын load test

100 Virtual User ашиглан хамгийн өндөр ачааллын туршилтыг 1 минутын турш хийсэн.

### Команд

```bash
k6 run --vus 100 --duration 1m script.js 2>&1 | tee results/run-100vu.txt
```

### Үр дүн

| Үзүүлэлт      |           Үр дүн |
| ------------- | ---------------: |
| VUs           |              100 |
| Duration      |            1 min |
| p(90)         |        237.87 ms |
| p(95)         |    **300.55 ms** |
| Average       |        159.38 ms |
| Minimum       |         53.95 ms |
| Maximum       |        695.70 ms |
| Throughput    | **148.04 req/s** |
| HTTP requests |             9018 |
| Iterations    |             4509 |
| Error rate    |        **0.00%** |
| Threshold     |         **PASS** |

100 VU-ийн үед throughput **148.04 req/s** хүрсэн бөгөөд error rate 0.00% хэвээр байсан.

p(95) latency нь 300.55 ms байсан тул 406.08 ms-ийн SLO threshold-ийг хангаж PASS болсон.

---

# 12. Нэгдсэн туршилтын үр дүн

Шинэ `results/*.txt` файлуудын бодит output-оос:

| Туршилт        |         VU |    Duration |         p(95) |       Throughput | Error rate | Threshold |
| -------------- | ---------: | ----------: | ------------: | ---------------: | ---------: | --------- |
| Baseline       |          1 | 1 iteration | **282.41 ms** |       0.82 req/s |      0.00% | PASS      |
| Stages         | 5→30→100→0 |       1 min | **335.16 ms** |  **45.93 req/s** |      0.00% | PASS      |
| Threshold PASS |         30 |       1 min | **402.98 ms** |  **39.12 req/s** |      0.00% | PASS      |
| Threshold FAIL |         30 |       1 min | **384.88 ms** |  **39.99 req/s** |      0.00% | FAIL*     |
| 5 VU           |          5 |       1 min | **349.72 ms** |   **6.67 req/s** |      0.00% | PASS      |
| 30 VU          |         30 |       1 min | **239.09 ms** |  **45.41 req/s** |      0.00% | PASS      |
| 100 VU         |        100 |       1 min | **300.55 ms** | **148.04 req/s** |      0.00% | PASS      |

`*` Threshold FAIL тестэд `p(95)<50ms` гэсэн зориудаар хатуу босго ашигласан.

---

# 13. Latency шинжилгээ

Шинэ туршилтын үр дүнгээс:

```text
5 VU       → 349.72 ms
30 VU      → 239.09 ms
100 VU     → 300.55 ms
Stages     → 335.16 ms
```

гэсэн p(95) latency-ууд гарсан.

Бүх үндсэн load test-ийн p(95) утгууд нь 406.08 ms-ийн SLO threshold-ээс бага байсан.

Threshold PASS тестийн p(95) нь **402.98 ms** байсан бөгөөд SLO threshold-д хамгийн ойр үр дүн гарсан.

---

# 14. Throughput шинжилгээ

Throughput нь систем нэг секундэд хэдэн HTTP request боловсруулж байгааг харуулна.

Шинэ load test-ийн үр дүн:

```text
5 VU       → 6.67 req/s
30 VU      → 45.41 req/s
100 VU     → 148.04 req/s
```

100 VU-ийн үед хамгийн өндөр throughput буюу **148.04 req/s** хэмжигдсэн.

Stages тестийн нийт throughput **45.93 req/s** байсан.

Threshold PASS болон FAIL тестүүдийн throughput ойролцоо:

```text
Threshold PASS → 39.12 req/s
Threshold FAIL → 39.99 req/s
```

байсан.

---

# 15. Error Rate шинжилгээ

Бүх туршилтын үед HTTP request-ийн error rate:

```text
0.00%
```

байсан.

SLO:

```text
error rate < 1%
```

учраас бүх туршилтад error rate threshold PASS болсон.

Threshold FAIL тестэд ч HTTP error rate 0.00% байсан. Тус тестийн FAIL нь HTTP error-оос бус, зөвхөн `p(95)<50ms` threshold биелээгүйтэй холбоотой.

---

# 16. PASS / FAIL дүн

## Үндсэн SLO

```text
p(95) < 406.08 ms
error rate < 1%
```

| Туршилт        | p(95) SLO | Error Rate SLO |
| -------------- | --------- | -------------- |
| Baseline       | PASS      | PASS           |
| Stages         | PASS      | PASS           |
| Threshold PASS | PASS      | PASS           |
| 5 VU           | PASS      | PASS           |
| 30 VU          | PASS      | PASS           |
| 100 VU         | PASS      | PASS           |

## Intentional FAIL

Threshold FAIL тест:

```text
Threshold: p(95) < 50 ms
Actual:    p(95) = 384.88 ms
Result:    FAIL
```

Энэ нь threshold-ийн FAIL ажиллагааг шалгах зорилготой зориудын тест юм.

---

# 17. Screenshot-ууд

Туршилтын terminal output-уудын screenshot-уудыг `screenshots/` хавтсанд байршуулсан.

```text
screenshots/
├── 01-baseline.png
├── 02-stages.png
├── 03-threshold-pass.png
├── 04-threshold-fail.png
├── 05-5vu.png
├── 06-30vu.png
└── 07-100vu.png
```

### Screenshot-ийн тайлбар

1. `01-baseline.png` — Baseline test
2. `02-stages.png` — 5 → 30 → 100 → 0 VU stages test
3. `03-threshold-pass.png` — Threshold PASS
4. `04-threshold-fail.png` — Intentional Threshold FAIL
5. `05-5vu.png` — 5 VU / 1 minute
6. `06-30vu.png` — 30 VU / 1 minute
7. `07-100vu.png` — 100 VU / 1 minute

---

# 18. Үр дүнгийн файлууд

Бүх туршилтын бүрэн k6 output-уудыг `results/` хавтсанд хадгалсан.

```text
results/
├── run-baseline.txt
├── run-stages.txt
├── run-threshold-pass.txt
├── run-threshold-fail.txt
├── run-05vu.txt
├── run-30vu.txt
└── run-100vu.txt
```

Файл бүр нь тухайн туршилтын бүрэн terminal output болон threshold-ийн үр дүнг агуулна.

---

# 19. Төслийн бүтэц

```text
lab2-k6/
│
├── README.md
├── script.js
├── stages.js
├── threshold-pass.js
├── threshold-fail.js
│
├── results/
│   ├── run-baseline.txt
│   ├── run-stages.txt
│   ├── run-threshold-pass.txt
│   ├── run-threshold-fail.txt
│   ├── run-05vu.txt
│   ├── run-30vu.txt
│   └── run-100vu.txt
│
└── screenshots/
    ├── 01-baseline.png
    ├── 02-stages.png
    ├── 03-threshold-pass.png
    ├── 04-threshold-fail.png
    ├── 05-5vu.png
    ├── 06-30vu.png
    └── 07-100vu.png
```

---

# 20. Ашигласан командууд

### k6 хувилбар шалгах

```bash
k6 version
```

### Baseline

```bash
k6 run script.js 2>&1 | tee results/run-baseline.txt
```

### Stages

```bash
k6 run stages.js 2>&1 | tee results/run-stages.txt
```

### Threshold PASS

```bash
k6 run threshold-pass.js 2>&1 | tee results/run-threshold-pass.txt
```

### Threshold FAIL

```bash
k6 run threshold-fail.js 2>&1 | tee results/run-threshold-fail.txt
```

### 5 VU / 1 minute

```bash
k6 run --vus 5 --duration 1m script.js 2>&1 | tee results/run-05vu.txt
```

### 30 VU / 1 minute

```bash
k6 run --vus 30 --duration 1m script.js 2>&1 | tee results/run-30vu.txt
```

### 100 VU / 1 minute

```bash
k6 run --vus 100 --duration 1m script.js 2>&1 | tee results/run-100vu.txt
```

---

# 21. Дүгнэлт

Энэхүү лабораторийн ажлын хүрээнд Grafana k6 ашиглан `test.k6.io` систем дээр performance testing хийж, latency, throughput болон error rate хэмжүүрүүдийг туршсан.

Шинэ baseline туршилтаар p(95) latency **282.41 ms**, error rate **0.00%** гарсан.

1 минутын load test-үүдийн үр дүн:

```text
5 VU   → p(95) = 349.72 ms,  throughput = 6.67 req/s
30 VU  → p(95) = 239.09 ms,  throughput = 45.41 req/s
100 VU → p(95) = 300.55 ms,  throughput = 148.04 req/s
```

Бүх үндсэн load test-д HTTP error rate **0.00%** байсан.

Stages тестийн үед ачаалал 5 VU-ээс 100 VU хүртэл нэмэгдэж, дараа нь буурсан бөгөөд нийт p(95) latency **335.16 ms**, throughput **45.93 req/s** гарсан.

Threshold PASS тестийн p(95) нь **402.98 ms** байсан бөгөөд 406.08 ms-ийн SLO threshold-ийг бага хэмжээгээр хангаж PASS болсон.

Мөн threshold механизмын FAIL ажиллагааг шалгахын тулд `p(95)<50ms` гэсэн зориудаар хатуу threshold ашигласан. Энэ туршилтаар p(95) **384.88 ms** гарсан тул threshold FAIL болсон. HTTP error rate харин **0.00%** хэвээр байсан.

Ингэснээр лабораторийн ажлын хүрээнд:

* Latency хэмжих
* p(95) threshold ашиглах
* Throughput хэмжих
* Error rate шалгах
* SLO тодорхойлох
* PASS threshold шалгах
* FAIL threshold шалгах
* Өөр өөр VU түвшний load test хийх

зэрэг шаардлагуудыг Grafana k6 ашиглан гүйцэтгэсэн.

---