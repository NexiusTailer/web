---
title: fmatch
description: Find a filename matching a pattern.
tags: []
---

:::warning

This function starts with lowercase letter.

:::

## คำอธิบาย

Find a filename matching a pattern.

| Name          | Description                                                    |
| ------------- | -------------------------------------------------------------- |
| name          | The string to hold the result in, returned as a packed string. |
| const pattern | The pattern that should be matched. May contain wildcards.     |
| index         | The number of the file, in case there are multiple matches.    |
| size          | The maximum size of parameter name                             |

## ส่งคืน

true on success, false on failure

## บันทึก

:::warning

This function does not work in the current SA:MP version!

:::

## ฟังก์ชั่นที่เกี่ยวข้องกัน

- [fopen](fopen): Open a file.
- [fclose](fclose): Close a file.
- [ftemp](ftemp): Create a temporary file stream.
- [fremove](fremove): Remove a file.
- [fwrite](fwrite): Write to a file.
- [fread](fread): Read a file.
- [fputchar](fputchar): Put a character in a file.
- [fgetchar](fgetchar): Get a character from a file.
- [fblockwrite](fblockwrite): Write blocks of data into a file.
- [fblockread](fblockread): Read blocks of data from a file.
- [fseek](fseek): Jump to a specific character in a file.
- [flength](flength): Get the file length.
- [fexist](fexist): Check, if a file exists.
- [fmatch](fmatch): Check, if patterns with a file name matches.
