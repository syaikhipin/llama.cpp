# llama.cpp

Inference of [Facebook's LLaMA](https://github.com/facebookresearch/llama) model in pure C/C++

**TEMPORARY NOTICE:**
Currently the quantized models run **only** on Apple Silicon. On other architectures, you can [use the F16 models](https://github.com/ggerganov/llama.cpp/issues/2#issuecomment-1464615286), but they will be much slower. Support will be [added later](https://github.com/ggerganov/ggml/pull/27)

## Description

The main goal is to run the model using 4-bit quantization on a MacBook.

- Plain C/C++ implementation without dependencies
- Apple silicon first-class citizen - optimized via Arm Neon and Accelerate framework
- Mixed F16 / F32 precision
- 4-bit quantization support
- Runs on the CPU

This was hacked in an evening - I have no idea if it works correctly.

Here is a typical run using LLaMA-7B:

```java
make -j && ./main -m ./models/7B/ggml-model-q4_0.bin -p "Building a website can be done in 10 simple steps:" -t 8 -n 512
I llama.cpp build info:
I UNAME_S:  Darwin
I UNAME_P:  arm
I UNAME_M:  arm64
I CFLAGS:   -I.              -O3 -DNDEBUG -std=c11   -fPIC -pthread -DGGML_USE_ACCELERATE
I CXXFLAGS: -I. -I./examples -O3 -DNDEBUG -std=c++11 -fPIC -pthread
I LDFLAGS:   -framework Accelerate
I CC:       Apple clang version 14.0.0 (clang-1400.0.29.202)
I CXX:      Apple clang version 14.0.0 (clang-1400.0.29.202)

make: Nothing to be done for `default'.
main: seed = 1678486056
llama_model_load: loading model from './models/7B/ggml-model-q4_0.bin' - please wait ...
llama_model_load: n_vocab = 32000
llama_model_load: n_ctx   = 512
llama_model_load: n_embd  = 4096
llama_model_load: n_mult  = 256
llama_model_load: n_head  = 32
llama_model_load: n_layer = 32
llama_model_load: n_rot   = 128
llama_model_load: f16     = 2
llama_model_load: n_ff    = 11008
llama_model_load: ggml ctx size = 4529.34 MB
llama_model_load: memory_size =   512.00 MB, n_mem = 16384
llama_model_load: .................................... done
llama_model_load: model size =  4017.27 MB / num tensors = 291

main: prompt: 'Building a website can be done in 10 simple steps:'
main: number of tokens in prompt = 15
     1 -> ''
  8893 -> 'Build'
   292 -> 'ing'
   263 -> ' a'
  4700 -> ' website'
   508 -> ' can'
   367 -> ' be'
  2309 -> ' done'
   297 -> ' in'
 29871 -> ' '
 29896 -> '1'
 29900 -> '0'
  2560 -> ' simple'
  6576 -> ' steps'
 29901 -> ':'

sampling parameters: temp = 0.800000, top_k = 40, top_p = 0.950000


Building a website can be done in 10 simple steps:
1) Select a domain name and web hosting plan
2) Complete a sitemap
3) List your products
4) Write product descriptions
5) Create a user account
6) Build the template
7) Start building the website
8) Advertise the website
9) Provide email support
10) Submit the website to search engines
A website is a collection of web pages that are formatted with HTML. HTML is the code that defines what the website looks like and how it behaves.
The HTML code is formatted into a template or a format. Once this is done, it is displayed on the user's browser.
The web pages are stored in a web server. The web server is also called a host. When the website is accessed, it is retrieved from the server and displayed on the user's computer.
A website is known as a website when it is hosted. This means that it is displayed on a host. The host is usually a web server.
A website can be displayed on different browsers. The browsers are basically the software that renders the website on the user's screen.
A website can also be viewed on different devices such as desktops, tablets and smartphones.
Hence, to have a website displayed on a browser, the website must be hosted.
A domain name is an address of a website. It is the name of the website.
The website is known as a website when it is hosted. This means that it is displayed on a host. The host is usually a web server.
A website can be displayed on different browsers. The browsers are basically the software that renders the website on the user’s screen.
A website can also be viewed on different devices such as desktops, tablets and smartphones. Hence, to have a website displayed on a browser, the website must be hosted.
A domain name is an address of a website. It is the name of the website.
A website is an address of a website. It is a collection of web pages that are formatted with HTML. HTML is the code that defines what the website looks like and how it behaves.
The HTML code is formatted into a template or a format. Once this is done, it is displayed on the user’s browser.
A website is known as a website when it is hosted

main: mem per token = 14434244 bytes
main:     load time =  1332.48 ms
main:   sample time =  1081.40 ms
main:  predict time = 31378.77 ms / 61.41 ms per token
main:    total time = 34036.74 ms
```

And here is another demo of running both LLaMA-7B and [whisper.cpp](https://github.com/ggerganov/whisper.cpp) on a single M1 Pro MacBook:

https://user-images.githubusercontent.com/1991296/224442907-7693d4be-acaa-4e01-8b4f-add84093ffff.mp4

## Usage

Here are the step for the LLaMA-7B model:

```bash
# build this repo
git clone https://github.com/ggerganov/llama.cpp
cd llama.cpp
make

# obtain the original LLaMA model weights and place them in ./models
ls ./models
65B 30B 13B 7B tokenizer_checklist.chk tokenizer.model

# install Python dependencies
python3 -m pip install torch numpy sentencepiece

# convert the 7B model to ggml FP16 format
python3 convert-pth-to-ggml.py models/7B/ 1

# quantize the model to 4-bits
./quantize ./models/7B/ggml-model-f16.bin ./models/7B/ggml-model-q4_0.bin 2

# run the inference
./main -m ./models/7B/ggml-model-q4_0.bin -t 8 -n 128
```

For the bigger models, there are a few extra quantization steps. For example, for LLaMA-13B, converting to FP16 format
will create 2 ggml files, instead of one:

```bash
ggml-model-f16.bin
ggml-model-f16.bin.1
```

You need to quantize each of them separately like this:

```bash
./quantize ./models/13B/ggml-model-f16.bin   ./models/13B/ggml-model-q4_0.bin 2
./quantize ./models/13B/ggml-model-f16.bin.1 ./models/13B/ggml-model-q4_0.bin.1 2
```

Everything else is the same. Simply run:

```bash
./main -m ./models/13B/ggml-model-q4_0.bin -t 8 -n 128
```

The number of files generated for each model is as follows:

```
7B  -> 1 file
13B -> 2 files
33B -> 4 files
65B -> 8 files
```

When running the larger models, make sure you have enough disk space to store all the intermediate files.

## Limitations

- Not sure if my tokenizer is correct. There are a few places where we might have a mistake:
  - https://github.com/ggerganov/llama.cpp/blob/26c084662903ddaca19bef982831bfb0856e8257/convert-pth-to-ggml.py#L79-L87
  - https://github.com/ggerganov/llama.cpp/blob/26c084662903ddaca19bef982831bfb0856e8257/utils.h#L65-L69
  In general, it seems to work, but I think it fails for unicode character support. Hopefully, someone can help with that
- I don't know yet how much the quantization affects the quality of the generated text
- Probably the token sampling can be improved
- x86 quantization support [not yet ready](https://github.com/ggerganov/ggml/pull/27). Basically, you want to run this on Apple Silicon. For now, on Linux and Windows you can use the F16 `ggml-model-f16.bin` model, but it will be much slower.
