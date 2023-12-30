WDM-KS Host API Notes
---------------------

Status History
--------------
January 16, 2011:
Added support for the WaveRT device API (Vista and newer) for even lower latency support.

November 10, 2005:
Made the following changes:* OpenStream: Try all PaSampleFormats internally if the selected format is not natively supported. This fixed several problems with sound cards 
that did not like using 24-bit or 3-byte formats.* OpenStream: Make the minimum framesPerHostIBuffer (and framesPerHostOBuffer) the default frame size for the playback/recording pin.*
ProcessingThread: Added a switch to only call PaUtil_EndBufferProcessing when the total input frames equals the total output frames.

September 5, 2004:
This is the first public release of the code. It should be considered an alpha release with a zero guarantee not to crash on any particular system. So far, it has only been tested in 
the author's development environment, which is a Win2k/SP2 PIII laptop with integratedSoundMAX driver and USB Tascam US-428, compiled with both MinGW(GCC 3.3) and MSVC++6 using the 
MS DirectX 9 SDK.It has been tested most extensively with the MinGW build, with most of the test programs (especially paqa_devs and paqa_errs) passing. There are a few notable
failures: patest_out_underflow and both of the blocking I/O tests (as blocking I/O is not implemented).At this point, the code needs to be tested with a much wider variety of 
configurations and feedback from testers regarding both working and failing cases.

What is the WDM-KS Host API?
----------------------------
PortAudio for Windows currently has three functional host implementations. MME uses the oldest Windows audio API, which does not provide good playback/recording latency. DirectX improves
this, but still imposes a penalty of tens of milliseconds due to system mixing of streams from multiple applications. ASIO offers very good latency, but requires special drivers that
are not always available for cheaper audio hardware. Alternatively, there are a number of free (but closed-source) ASIO implementations that connect to the lower-level Windows
"kernel streaming" API, but again these require special installation by the user, and can be limited in functionality or difficult to use.

This is where the PortAudio "WDM-KS" host implementation comes in. It connects PortAudio directly to the same kernel streaming API that these ASIO bridges use. This avoids the DirectX
mixing penalty, giving at least as good latency as any ASIO driver, but has the advantage of working with any Windows audio hardware available through the normal MME/DirectX routes, without
requiring the user to install any additional device drivers, and allowing all device selection to be done through the normal PortAudio API.
Note that in general, you should only use this host API if your application has a real need for very low latency audio (<20 ms), either because you are generating sounds in real time
based on user input, or because you are processing recorded audio in real time.
The only thing to be aware of is that using the KS interface will block that device from being used by the rest of the system through the higher-level APIs, or conversely, if the system
is using a device, the KS API will not be able to use it. MS recommends that you should only keep the device open when your application has focus.In PortAudio terms, this means having a
stream open on a WDMKS device.

How to use
----------
To add the WDMKS backend to your program that already usesPortAudio, you need to define PA_USE_WDMKS=1 in your build file.and include pa_win_wdmks\pa_win_wdmks.c in your build.The file
should compile in both C and C++. You will need a DirectX SDK installed on your system for theks.h and ksmedia.h header files.You will need to link to the system "setupapi" library. 
Note that if you are using MinGW, you will get more warnings from the DX header files when using GCC (C), and still a few warnings with G++(CPP).
