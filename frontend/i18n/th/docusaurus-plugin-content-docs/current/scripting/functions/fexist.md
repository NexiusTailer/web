---
title: fexist
sidebar_label: fexist
description: Checks if a specific file exists in the scriptfiles directory.
tags: []
---

:::warning

This function starts with lowercase letter.

:::

## คำอธิบาย

Checks if a specific file exists in the scriptfiles directory.

| Name        | Description                                            |
| ----------- | ------------------------------------------------------ |
| pattern[]   | The name of the file, optionally containing wild-cards |
| characters. |

## ส่งคืน

The number of files that match the pattern.

## ตัวอย่าง

```c
// Check, if "file.txt" exists
if (fexist("file.txt"))
{
    // Success

    // Print the success
    print("\"file.txt\" exists.");
}
else
{
    // Error
    print("\"file.txt\" does not exist.");
}
```

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
