# Denon Daemon

My Denon receiver has a serial port.  This serial port can be used
to control and report on what the receiver is doing.  So if I send it
"PW?" then it will report on the power state.

There's protocol documentation [on the Denon site](https://usa.denon.com/us/product/hometheater/receivers/avr3808ci?docname=AVR-3808CISerialProtocol_Ver520a.pdf) (pdf)

This daemon (written for MacOS) will listen for commands on a pipe and
communicate with the receiver and write out status entries so it is easy
for a program to see what is going on.

I've hard coded the input values, so it displays names that are
meaningful to me.

    % denon
              Input: TIVO
       MainSurround: DOLBY PL2 C
           MainZone: ON
               Mute: OFF
              Power: ON
             Volume: 55 -25.0dB
             Z2Mute: OFF
           Z2Volume: 99 ---.-dB
              Zone2: OFF
         Zone2Input: SOURCE

    % denon input mac
              Input: MAC
       MainSurround: 5CH STEREO
           MainZone: ON
               Mute: OFF
              Power: ON
             Volume: 55 -25.0dB
             Z2Mute: OFF
           Z2Volume: 99 ---.-dB
              Zone2: OFF
         Zone2Input: SOURCE


It also writes out a log file.

    % tail /var/log/denon 
    Wed Nov 14 18:30:13 2018 status.MainSurround = DOLBY PL2 C
    Wed Nov 14 18:36:37 2018 status.MainSurround = DOLBY PL2 C
    Wed Nov 14 18:40:48 2018 status.MainSurround = DOLBY PL2 C
    Wed Nov 14 18:41:14 2018 status.MainSurround = DOLBY PL2 C
    Wed Nov 14 18:41:25 2018 status.MainSurround = DOLBY PL2 C
    Wed Nov 14 19:08:36 2018 Wrote SIVCR
    Wed Nov 14 19:08:37 2018 status.Input = MAC
    Wed Nov 14 19:08:37 2018 status.MainSurround = 5CH STEREO
    Wed Nov 14 19:08:41 2018 status.MainSurround = 5CH STEREO
    Wed Nov 14 19:08:42 2018 status.MainSurround = 5CH STEREO

## Method of operation

There are basically two threads, which I run as processes ('cos this is
perl).  One thread just reads the output from the receiver and updates
the status file.  The other thread reads input from the pipe, works out
the necessary Denon protocol command and sends it to the receiver.  These
two processes effectively work asynchronously.

What this means is that if I change the input using the normal remote control
then the denon reports this change and the code picks it up.

On startup it sends a bunch of query commands to the receiver, just so
the current state can be determined.
