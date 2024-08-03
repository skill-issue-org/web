---
layout: post
category: challenge
title: '[Weekly Challenge] Stasiun Balapan'
---

```
Points: 6
Author: dwisiswant0
```

## Description

Jarak tempuh untuk mencapai stasiun adalah **N** kilometer, dan Ikhsan memutuskan untuk mengambil rute tercepat dengan melewati jalan tikus, tetapi ketika langkahnya ganjil ia memilih untuk berisirahat dengan _nyebat duls_ karena penyakit yang diidapnya, _myalgic encephalomyelitis_ (ME) atau sindrom kelelahan kronis yang menyebabkan tubuhnya cepat lelah karena kebiasaannya yang buruk, yaitu ngocok lima waktu.

## Hints

`N=100`

## Steps to Resolve

To retrieve the flag from the **getFlag** function ([main.go:72](https://github.com/skill-issue-org/challenges/blob/master/stasiun-balapan/src/main.go#L72)), the global variable **attempt** must be greater than or equal to the integer value produced by [`rand.Intn`](https://pkg.go.dev/math/rand#Intn). Given the hint that **N** is **100**, the result of `rand.Intn(N-50) + 50` ([main.go:75](https://github.com/skill-issue-org/challenges/blob/master/stasiun-balapan/src/main.go#L75)) is a non-negative pseudo-random number ranging from **50** to **100**.

To increment **attempt**, the **masuk** function must be invoked. This invocation is conditional ([main.go:52](https://github.com/skill-issue-org/challenges/blob/master/stasiun-balapan/src/main.go#L52)), requiring that the variable **i** be an odd number when divided by **2** in order to call the function using a goroutine.

Since the maximum range of the pseudo-random number is **100**, the maximum number of attempts should be `N * 2 = 200`.

```bash
$ yes | head -n 200 | /path/to/stasiun-balapan
Stasiun Balapan Solo
Masukin: 
w0w! nih, flagnya: n1ng$etasiyun₿alapan
```

The flag is returned.