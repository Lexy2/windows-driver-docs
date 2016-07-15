---
Description: 'A Wave Sink for Kernel-Mode Software Synthesizers'
MS-HAID: 'audio.a\_wave\_sink\_for\_kernel\_mode\_software\_synthesizers'
MSHAttr: 'PreferredLib:/library/windows/hardware'
title: 'A Wave Sink for Kernel-Mode Software Synthesizers'
---

# A Wave Sink for Kernel-Mode Software Synthesizers


## <span id="a_wave_sink_for_kernel_mode_software_synthesizers"></span><span id="A_WAVE_SINK_FOR_KERNEL_MODE_SOFTWARE_SYNTHESIZERS"></span>


As explained in [Synthesizers and Wave Sinks](synthesizers-and-wave-sinks.md), the DMus port driver implements the wave sink for a software synthesizer that operates in kernel mode. The miniport driver for the synthesizer exposes an [ISynthSinkDMus](audio.isynthsinkdmus) interface to the port driver. The port driver's wave sink uses this interface to read the wave data that is produced by the synthesizer.

To make use of the DMus port driver's wave sink, a DMus miniport driver should define a DirectMusic filter with two types of pin:

-   A DirectMusic input pin or MIDI input pin. This pin is a sink for a render stream containing MIDI messages.

-   A wave output pin. This pin is a source for a render stream containing PCM samples.

The following figure shows a DirectMusic filter containing a synthesizer node ([**KSNODETYPE\_SYNTHESIZER**](audio.ksnodetype_synthesizer)). This filter meets the preceding requirements for a kernel-mode software synthesizer by providing a DirectMusic input pin and a wave output pin. (In addition, a DMus miniport driver that supports legacy MIDI synthesis can provide a MIDI input pin.)

![diagram illustrating a directmusic filter for a kernel-mode software synthesizer](images/wavesink.png)

On the left side of the figure, a MIDI stream enters the filter through the DirectMusic input pin. This pin has an [IMXF](audio.imxf) interface that it exposes to the port driver. The port driver obtains this interface by calling the [**IMiniportDMus::NewStream**](audio.iminiportdmus_newstream) method. The port driver feeds MIDI messages to the pin by calling the [**IMXF::PutMessage**](audio.imxf_putmessage) method.

On the right side of the figure, a wave stream exits the filter through the wave output pin and flows to the port driver's wave sink. The port driver communicates with the pin through its **ISynthSinkDMus** interface. The port driver obtains this interface by first calling **IMiniportDMus::NewStream** to obtain a stream object with an **IMXF** interface, and then querying the object for its **ISynthSinkDMus** interface. The wave sink pulls wave data from the pin by calling the [**ISynthSinkDMus::Render**](audio.isynthsinkdmus_render) method.

Although a hardware synthesizer could, in principle, rely on the port driver's wave sink for rendering, the call to **ISynthSinkDMus::Render** adds enough latency to the MIDI stream to make it unattractive for many interactive applications. To reduce stream latency, hardware synthesizers are likely to have internal connections to mixing and wave-rendering hardware instead of using the port driver's wave sink. This type of synthesizer replaces the wave output pin on the right side of the preceding figure with a hardwired connection (represented as a [*bridge pin*](wdkgloss.b#wdkgloss-bridge-pin)) to a hardware mixer.

The **ISynthSinkDMus** interface provides methods to render wave data through a wave sink, convert from reference time to sample time and back, and synchronize to the master clock:

[**ISynthSinkDMus::RefTimeToSample**](audio.isynthsinkdmus_reftimetosample)

[**ISynthSinkDMus::Render**](audio.isynthsinkdmus_render)

[**ISynthSinkDMus::SampleToRefTime**](audio.isynthsinkdmus_sampletoreftime)

[**ISynthSinkDMus::SyncToMaster**](audio.isynthsinkdmus_synctomaster)

**ISynthSinkDMus** inherits from the **IMXF** interface. For more information, see [ISynthSinkDMus](audio.isynthsinkdmus).

The DMus miniport driver in the preceding figure identifies its DirectMusic input pin and wave output pin as follows:

-   To identify its DirectMusic input pin, the miniport driver defines the pin's data range to have a major format of type KSDATAFORMAT\_TYPE\_MUSIC and a subformat of type KSDATAFORMAT\_SUBTYPE\_DIRECTMUSIC. This combination indicates that the pin accepts a time-stamped MIDI stream. The data range descriptor is a structure of type [**KSDATARANGE\_MUSIC**](audio.ksdatarange_music). (For an example, see [DirectMusic Stream Data Range](directmusic-stream-data-range.md).) The miniport driver defines the pin's data flow direction to be KSPIN\_DATAFLOW\_IN. (The [**PCPIN\_DESCRIPTOR**](audio.pcpin_descriptor) structure's **KsPinDescriptor**.DataFlow member indicates the data flow direction.) When calling **IMiniportDMus::NewStream** to create the stream object for this pin, the port driver sets the *StreamType* parameter to DMUS\_STREAM\_MIDI\_RENDER.

-   To identify its wave output pin, the miniport driver defines the pin's data range to have a major format of type KSDATAFORMAT\_TYPE\_AUDIO and a subformat of type KSDATAFORMAT\_SUBTYPE\_PCM. This combination indicates that the pin emits a wave audio stream containing PCM samples. The data range descriptor is a structure of type [**KSDATARANGE\_AUDIO**](audio.ksdatarange_audio). (See the example in [PCM Stream Data Range](pcm-stream-data-range.md).) The miniport driver defines the pin's data flow direction to be KSPIN\_DATAFLOW\_OUT. When calling **IMiniportDMus::NewStream** to create the stream object for this pin, the port driver sets the *StreamType* parameter to DMUS\_STREAM\_WAVE\_SINK.

In addition, if the driver were to support a MIDI input pin for the synthesizer, its definition would be similar to that of the DirectMusic input pin, but the pin definition would specify a subformat of type KSDATAFORMAT\_SUBTYPE\_MIDI, and the pin would accept a raw MIDI stream rather than a time-stamped MIDI stream.

 

 


--------------------
[Send comments about this topic to Microsoft](mailto:wsddocfb@microsoft.com?subject=Documentation%20feedback%20[audio\audio]:%20A%20Wave%20Sink%20for%20Kernel-Mode%20Software%20Synthesizers%20%20RELEASE:%20%287/14/2016%29&body=%0A%0APRIVACY%20STATEMENT%0A%0AWe%20use%20your%20feedback%20to%20improve%20the%20documentation.%20We%20don't%20use%20your%20email%20address%20for%20any%20other%20purpose,%20and%20we'll%20remove%20your%20email%20address%20from%20our%20system%20after%20the%20issue%20that%20you're%20reporting%20is%20fixed.%20While%20we're%20working%20to%20fix%20this%20issue,%20we%20might%20send%20you%20an%20email%20message%20to%20ask%20for%20more%20info.%20Later,%20we%20might%20also%20send%20you%20an%20email%20message%20to%20let%20you%20know%20that%20we've%20addressed%20your%20feedback.%0A%0AFor%20more%20info%20about%20Microsoft's%20privacy%20policy,%20see%20http://privacy.microsoft.com/en-us/default.aspx. "Send comments about this topic to Microsoft")

