---
title: fclose
sidebar_label: fclose
description: Closes a file.
tags: []
---

:::warning

This function starts with lowercase letter.

:::

## คำอธิบาย

Closes a file. Files should always be closed when the script no longer needs them (after reading/writing).

| Name        | Description                                  |
| ----------- | -------------------------------------------- |
| File:handle | The file handle to close. Returned by fopen. |

## ส่งคืน

1: The function executed successfully.

0: The function failed to execute. The file could not be closed. It may already be closed.

## ตัวอย่าง

```c
// Open "file.txt" in "append only" mode
new File:handle = fopen("file.txt", io_append);

// Check, if file is open
if (handle)
{
    // Success

    // Write "Hi there!" into the file
    fwrite(handle, "Hi there!");

    // Close the file
    fclose(handle);
}
else
{
    // Error
    print("Failed to open file \"file.txt\".");
}
```

## บันทึก

:::warning

Using an invalid handle will crash your server! Get a valid handle by using fopen or ftemp.

:::

## ฟังก์ชั่นที่เกี่ยวข้องกัน

- [fopen](fopen): Open a file.
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
