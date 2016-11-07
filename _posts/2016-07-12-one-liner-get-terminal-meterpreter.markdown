---
layout: post
title: "One-liner: get to terminal from meterpreter"
date: 2016-07-12
tags: netsec
---
    python -c 'import pty; pty.spawn("/bin/sh")'
