---
layout: post
category: challenge
title: '[Weekly Challenge] Ibu Kota Negara (Mangcrack)'
---

```
Points: 8
Author: dwisiswant0
```

## Description

Dibutuhkan kata sandi untuk melakukan simulasi tata ibu kota.

## Hints

`^[a-z]+$`

## Steps to Resolve

We need to crack the program so that it matches up with `vars.Password` ([main.go:39](https://github.com/skill-issue-org/challenges/blob/master/ibu-kota-negara/src/main.go#L39)) from the `skill-issue.org/ibu-kota-negara/pkg/vars` package. This should be done following the `^[a-z]+$` pattern that's given as a hint. This pattern implies that the password should only contain lowercase letters.

1. Download `rockyou.txt` common password list.
2. Execute the following command:

```bash
$ grep -Po "^[a-z]+$" /path/to/rockyou.txt | xargs -I % sh -c 'printf % | /path/to/ibu-kota-negara 2>/dev/null' | grep flag
Masukin passwordnya dulu: goks! ywdh nih flagnya: 1buK07a-Ne9@ra                       .|
```

The flag is returned.

But by doing so, we won't know what the right password is and the cracking process will take longer. You can use the following Bash script instead to print the password for when the program shows the flag, and execute it with threading.

```bash 
#!/bin/bash

function crack() {
  local passwd="${1}"

  local cmd="/path/to/ibu-kota-negara <<< '${passwd}' 2>/dev/null"
  local out=$(eval "${cmd}")

  local flag=$(grep -o "flag.*" <<< "${out}")

  [[ "${flag}" != "" ]] && echo "Password: ${passwd} | ${flag}"
}

THREAD=100

COUNT=1
grep -Po "^[a-z]+$" /path/to/rockyou.txt | while read -r line; do
  crack "${line}" &

  [[ "${COUNT}" -eq "${THREAD}" ]] && { wait; COUNT=0; } || ((COUNT++))
done
```

1. Execute the script:

```bash
$ bash crack.sh
Password: simcity | flagnya: 1buK07a-Ne9@ra
^C
```

The flag is returned by using what password.