---
title: 'How to fix Nodejs heap out of memory error?'
date: 2022-08-14T15:32:14Z
lastmod: '2022-08-14'
tags: ['nodejs']
draft: false
summary: 'How to fix nodejs heap out of memory error?'
layout: PostSimple
canonicalUrl: https://talwinder.tech/blog/persisting-ag-grid-state-in-angular
---

This problem occur when you run a program that consumers more heap memory than available.

Run the following command to see heap memory available for Nodejs:

```bash
node -e 'console.log(v8.getHeapStatistics().heap_size_limit/(1024*1024))'
```

To increase this limit set `--max-old-space-size` environment variable to how much you want to increase

## Increase heap memory size on macos and linux

```bash
export NODE_OPTIONS=--max-old-space-size=4096
```

## Increase heap memory size on windows

**Powershell**

```bash
$env:NODE_OPTIONS="--max-old-space-size=4096"
```

Run the following command again you should see a higher value now, indicating heap memory has been increased.

```bash
node -e 'console.log(v8.getHeapStatistics().heap_size_limit/(1024*1024))'
```

If you issue is not resolved even after setting limit to a higher value. You should check your application for any
possible memory leaks.