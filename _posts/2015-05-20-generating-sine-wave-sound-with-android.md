---
layout: post
title: GENERATING SINE WAVE SOUND WITH ANDROID
published: true
---

### Introduction
In the following post I will show and explain how to create a sine wave sound in Android.
The sine wave is a mathematical curve that describes a smooth repetitive oscillation. Sine wave sounds can be used in many fields, one of them being robotics. 

### Sine wave formula
The sine wave formula is:

<b>y(t) = A sin(2πft + ρ) = A sin(ωt + ρ)</b>

The above formula can be explained in sound terms as follows:

<b>y = amplitude x sin ( 2π ( velocity of rotation in cycles per second))</b>

Increasing the amplitude of the sine wave, how high the tops and bottoms of the wave go, increases the volume.  Increasing or decreasing the cycle rate, how many cycles over distance/time, increases and decreases the pitch of the sound – how high or low the tone sounds.

### Android implementation

To create a sine wave sound in Android we will use AudioTrack class. Using the Android Math.sin() function we can generate a pure sine wave in a memory buffer, then use the AudioTrack class to play the sine wave.

``` java
private void playSound(double frequency, int duration) {
    // AudioTrack definition
    int mBufferSize = AudioTrack.getMinBufferSize(44100,
    AudioFormat.CHANNEL_OUT_MONO,
    AudioFormat.ENCODING_PCM_8BIT);

    AudioTrack mAudioTrack = new AudioTrack(AudioManager.STREAM_MUSIC, 44100,
    AudioFormat.CHANNEL_OUT_MONO, AudioFormat.ENCODING_PCM_16BIT,
    mBufferSize, AudioTrack.MODE_STREAM);

    // Sine wave
    double[] mSound = new double[4410];
    short[] mBuffer = new short[duration];
    for (int i = 0; i < mSound.length; i++) {
        mSound[i] = Math.sin((2.0*Math.PI * i/(44100/frequency)));
        mBuffer[i] = (short) (mSound[i]*Short.MAX_VALUE);
    }

    mAudioTrack.setStereoVolume(AudioTrack.getMaxVolume(), AudioTrack.getMaxVolume());
    mAudioTrack.play();

    mAudioTrack.write(mBuffer, 0, mSound.length);
    mAudioTrack.stop();
    mAudioTrack.release();

}
```

### Code explanation
If we call the function above as follows: <b>play(1500, 44100)</b> the device will produce a 1500 Hz sound for one second (44100 is the equivalent of 1 second).

