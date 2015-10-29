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
The code is pretty self explanatory so we can move on to the next step.
Next I created a RecyclerView and an adapter as shown in a <a href="http://programminglife.io/coordinatorlayout-and-floatingactionbutton/" target="_blank"> previous post </a>. I will not go into UI details as you can find all the code on <a href="https://github.com/andreivisan/BluetoothPrinter" target="_blank"> my Github repository </a>.
Next I added the action that triggers the app to connect to the printer.

```java
mRecyclerView.addOnItemTouchListener(
        new RecyclerItemClickListener(this, new RecyclerItemClickListener.OnItemClickListener() {
            @Override
            public void onItemClick(View view, int position) {
                mDevice = mPairedDevices.get(position);
                openBtConnection();
            }
        })
);
        
//.......

private void openBtConnection() {
    try {
        // Standard SerialPortService ID
        UUID uuid = UUID.fromString("00001101-0000-1000-8000-00805f9b34fb");
        mSocket = mDevice.createRfcommSocketToServiceRecord(uuid);
        mSocket.connect();
        OutputStream outputStream = mSocket.getOutputStream();
        mOutputStream = new BufferedOutputStream(outputStream);
        mInputStream = mSocket.getInputStream();

        beginListenForData();

        Snackbar.make(mCoordinatorLayout, "Bluetooth connection opened", Snackbar.LENGTH_SHORT).show();
    } catch (Exception e) {
        Log.e(tag, e.getMessage(), e);
    }
}

private void beginListenForData() {
    try {
        final Handler handler = new Handler();

        // This is the ASCII code for a newline character
        final byte delimiter = 10;

        stopWorker = false;
        readBufferPosition = 0;
        readBuffer = new byte[1024];

        mWorkerThread = new Thread(new Runnable() {
            public void run() {
                while (!Thread.currentThread().isInterrupted() && !stopWorker) {
                    try {
                        int bytesAvailable = mInputStream.available();
                        if (bytesAvailable > 0) {
                            byte[] packetBytes = new byte[bytesAvailable];
                            mInputStream.read(packetBytes);
                            for (int i = 0; i < bytesAvailable; i++) {
                                byte readByte = packetBytes[i];
                                if (readByte == delimiter) {
                                    byte[] encodedBytes = new byte[readBufferPosition];
                                    System.arraycopy(readBuffer, 0,
                                                     encodedBytes, 0,
                                                     encodedBytes.length);
                                    final String data = new String(encodedBytes, 
                                                                   "US-ASCII");
                                    readBufferPosition = 0;
                                    handler.post(new Runnable() {
                                        public void run() {
                                            Snackbar.make(
                                                mCoordinatorLayout, 
                                                "Printer is ready", 
                                                Snackbar.LENGTH_SHORT
                                            ).show();
                                        }
                                    });
                                } else {
                                    readBuffer[readBufferPosition++] = readByte;
                                }
                            }
                        }
                    } catch (IOException ex) {
                        stopWorker = true;
                    }
                }
            }
        });
        mWorkerThread.start();
    } catch (Exception e) {
        Log.e(tag, e.getMessage(), e);
    }
}
```

As you can see in the code above all we did was to open a basic socket connection from the app to the Bluetooth printer.
Now we are ready to start printing so let's implement the Test Printer action.

``` java
mTestPrinter.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        sendDataToPrinter();
    }
});

//.......

private void sendDataToPrinter() {
    try {
        ESCPOSDriver escposDriver = new ESCPOSDriver();

        String msgLeft = "Left";
        msgLeft += "\n";
        String msgCenter = "Center";
        msgCenter += "\n";
        String msgRight = "Right";
        msgRight += "\n";

        //Initialize
        escposDriver.initPrint(mOutputStream);

        escposDriver.printLineAlignLeft(mOutputStream, msgLeft);
        escposDriver.printLineAlignCenter(mOutputStream, msgCenter);
        escposDriver.printLineAlignRight(mOutputStream, msgRight);

        escposDriver.flushCommand(mOutputStream);

        mOutputStream.flush();

        Snackbar.make(mCoordinatorLayout, "Data sent", Snackbar.LENGTH_SHORT).show();
    } catch (Exception e) {
        Log.e(tag, e.getMessage(), e);
    }
}
```

Again, all we did here was just to call the methods we have implement in the triver and send the correct byte arrays to the printer.
The only thing left to impement is to close the connection to the printer. For this we have another button, with an unfortunate name (I developed it in a hurry), Close printer, that will close the connection to the selected printer.

```java
mClosePrinter.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        closeBT();
    }
});

//.......

void closeBT() {
    try {
        stopWorker = true;
        mOutputStream.close();
        mInputStream.close();
        mSocket.close();

        Snackbar.make(mCoordinatorLayout, "Bluetooth closed", Snackbar.LENGTH_SHORT).show();
    } catch (Exception e) {
        Log.e(tag, e.getMessage(), e);
    }
}
```

Again, all the code to develop the complete app can be found on <a href="https://github.com/andreivisan/BluetoothPrinter" target="_blank"> my Github repository </a>. Please let me know in the comments section bellow if you have any questions.
