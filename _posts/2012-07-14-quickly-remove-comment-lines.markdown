---
layout: post
title:  "Quickly remove comment lines in linux text files"
date:   2012-1-21
categories: bash
---
Use sed to [quickly remove comment lines in linux text files.](http://soft.zoneo.net/Linux/remove_comment_lines.php)

    # If comment starts with #:
    sed '/^\#/d' myFile > tt 
    mv tt myFile
