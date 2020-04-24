# Fuzzer for libFLAC decoder

## Plugin Design Considerations
The fuzzer plugin for FLAC decoder is designed based on the understanding of the
codec and tries to achieve the following:

##### Maximize code coverage
The configuration parameters are not hardcoded, but instead selected based on
incoming data. This ensures more code paths are reached by the fuzzer.

FLAC supports the following two decoder API's for native mode:
1. FLAC__stream_decoder_process_single (frame by frame decoding)
2. FLAC__stream_decoder_process_until_end_of_stream (decoding entire stream)

One of these two decoder API's will be called based on LSB of 5th byte of data.

This also ensures that the plugin is always deterministic for any given input.

##### Maximize utilization of input data
The plugin feeds the input data to the codec using a read_callback as and when
requested by the decoder. The read_callback feeds the input data to the decoder
until the end of stream.

This ensures that the plugin tolerates any kind of input (empty, huge,
malformed, etc) and doesnt `exit()` on any input and thereby increasing the
chance of identifying vulnerabilities.

## Build

This describes steps to build flac_dec_fuzzer binary.

### Android

#### Steps to build
Build the fuzzer
```
  $ mm -j$(nproc) flac_dec_fuzzer
```

#### Steps to run
Create a directory CORPUS_DIR and copy some flac files to that folder
Push this directory to device.

To run on device
```
  $ adb sync data
  $ adb shell /data/fuzz/arm64/flac_dec_fuzzer/flac_dec_fuzzer CORPUS_DIR
```
To run on host
```
  $ $ANDROID_HOST_OUT/fuzz/x86_64/flac_dec_fuzzer/flac_dec_fuzzer CORPUS_DIR
```

## References:
 * http://llvm.org/docs/LibFuzzer.html
 * https://github.com/google/oss-fuzz
