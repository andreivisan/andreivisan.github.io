---
layout: post
title: ANDROID EPSON PRINTER CONNECTION
published: false
---

### Introduction

In this post I will cover the steps to connect to any type of Bluetooth printer that is based on ESC/POS (Epson) protocol. 
In the end we will have an app looking like the image bellow, that lists all the paired devices via the Bluetooth menu in the Settings.

![Bluetooth printer app](/public/images/bt_printer.png)

When clicking on the printer name in the list, the app will open a socket connection with the printer. Then by clicking on the 
Test Printer you can print 3 rows, one left aligned, one centered and one right aligned.

### The code

I started by creating a simple Android project with an empty activity. Next I implemented the ESC/POS driver as shown bellow:

``` java
public class ESCPOSDriver {

    private static String tag = ESCPOSDriver.class.getSimpleName();

    private static final byte[] LINE_FEED = {0x0A};
    private static final byte[] PAPER_FEED = {27, 0x4A, (byte)0xFF};
    private static final byte[] PAPER_CUT = {0x1D, 0x56, 0x1};
    private static final byte[] ALIGN_LEFT = {0x1B, 0x61, 0};
    private static final byte[] ALIGN_CENTER = {0x1B, 0x61, 1};
    private static final byte[] ALIGN_RIGHT = {0x1B, 0x61, 2};
    private static final byte[] BOLD_ON = {0x1B, 0x45, 1};
    private static final byte[] BOLD_OFF = {0x1B, 0x45, 0};
    private static final byte[] INIT = {0x1B, 0x40};
    private static final byte[] STANDARD_MODE = {0x1B, 0x53};
    private static final byte[] SWITCH_COMMAND = {0x1B, 0x69, 0x61, 0x00};
    private static final byte[] FLUSH_COMMAND = {(byte)0xFF, 0x0C};

    public void initPrint(BufferedOutputStream bufferedOutputStream) {
        try {
            bufferedOutputStream.write(SWITCH_COMMAND);
            bufferedOutputStream.write(INIT);
        } catch (IOException e) {
            Log.e(tag, e.getMessage(), e);
        }
    }

    public void printLineAlignLeft(BufferedOutputStream bufferedOutputStream, String lineData) {
        try {
            bufferedOutputStream.write(ALIGN_LEFT);
            bufferedOutputStream.write(lineData.getBytes());
            bufferedOutputStream.write(LINE_FEED);
        } catch (IOException e) {
            Log.e(tag, e.getMessage(), e);
        }
    }

    public void printLineAlignCenter(BufferedOutputStream bufferedOutputStream, String lineData) {
        try {
            bufferedOutputStream.write(ALIGN_CENTER);
            bufferedOutputStream.write(lineData.getBytes());
            bufferedOutputStream.write(LINE_FEED);
        } catch (IOException e) {
            Log.e(tag, e.getMessage(), e);
        }
    }

    public void printLineAlignRight(BufferedOutputStream bufferedOutputStream, String lineData) {
        try {
            bufferedOutputStream.write(ALIGN_RIGHT);
            bufferedOutputStream.write(lineData.getBytes());
            bufferedOutputStream.write(LINE_FEED);
        } catch (IOException e) {
            Log.e(tag, e.getMessage(), e);
        }
    }

    public void finishPrint(BufferedOutputStream bufferedOutputStream) {
        try {
            bufferedOutputStream.write(PAPER_FEED);
            bufferedOutputStream.write(PAPER_CUT);
        } catch (IOException e) {
            Log.e(tag, e.getMessage(), e);
        }
    }

    public void flushCommand(BufferedOutputStream bufferedOutputStream) {
        try {
            bufferedOutputStream.write(FLUSH_COMMAND);
            bufferedOutputStream.write(PAPER_FEED);
            bufferedOutputStream.write(PAPER_CUT);
        } catch (IOException e) {
            Log.e(tag, e.getMessage(), e);
        }
    }

}
```
