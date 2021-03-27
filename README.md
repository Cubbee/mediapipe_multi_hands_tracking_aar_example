This fork serves as a sample on how to receive Bitmap image from graph for additional postprocessing (https://github.com/google/mediapipe/issues/831).

It works with aar build with https://github.com/Cubbee/mediapipe, you can find instructions on how to build aar there.

Here is the outline on how to receive the bitmap from mediapipe graph:

1. From the mediapipe side, you need to add GpuBufferToImageFrameCalculator to the graph somewhere, for example `mediapipe/graphs/hand_tracking/hand_tracking_mobile.pbtxt`

```
node {
   calculator: "GpuBufferToImageFrameCalculator"
   input_stream: "throttled_input_video"
   output_stream: "throttled_input_video_cpu"
 }
```

2. Add this calculator to needed graph BUILD file `mediapipe/graphs/hand_tracking/BUILD`
```
cc_library(
    name = "mobile_calculators",
    deps = [
        "//mediapipe/calculators/core:constant_side_packet_calculator",
        "//mediapipe/calculators/core:flow_limiter_calculator",
        "//mediapipe/graphs/hand_tracking/subgraphs:hand_renderer_gpu",
        "//mediapipe/modules/hand_landmark:hand_landmark_tracking_gpu",
        "//mediapipe/gpu:gpu_buffer_to_image_frame_calculator",
    ],
)
```
3. Listen to the packet and decode it to bitmap
```
processor.addPacketCallback(
    "transformed_image_cpu"
) { packet ->
    println("Received image with ts: ${packet.timestamp}")
    val image = AndroidPacketGetter.getBitmapFromRgba(packet)
}
```

Image is received from the graph and saved to external storage folder `mediapipe` as an example.

You need to give the app the external storage permission manually through the phone settings.

Don't forget to run `adb shell setprop log.tag.MainActivity VERBOSE` to enable logging