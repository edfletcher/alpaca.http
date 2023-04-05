# Alpaca.http

Adds an HTTP server with a very simple REST interface.

### Interface

`POST /prompt` with the prompt given in the request body as `text/plain`. Returns the prompt's ID as `text/plain`.

`GET /prompt/:id` with a prompt `:id` to retrieve the prompt & response as `application/json`. If the response is still pending, will return HTTP code 202 with no body. If the `:id` is not valid, returns HTTP 404.

### Example
```shell
$ curl -d "tell me about alpacas" http://127.0.0.1:42000/prompt
1e18e1d50423fa29
$ curl -v http://127.0.0.1:42000/prompt/1e18e1d50423fa29
...
< HTTP/1.1 202 Accepted
...
$ sleep 30
$ curl -s http://127.0.0.1:42000/prompt/1e18e1d50423fa29 | jq .
{
  "prompt": "tell me about alpacas",
  "response": "Alpacas are small to medium-sized animals native to South America. They belong to the family Camelidae and genus Vicugna, which is composed of three species -Vicugna vicua (South American camelid), V. guadaloupensis (Guadalupe alpaca) ,and V. pacu(Peruvian alpaca). Alpacas are raised primarily for their fleece and meat, but they also have been bred as companion animals in some parts of the world. They typically live between 8 to 10 years in captivity, although one record-holding female lived up to 23 years.",
  "elapsed_ms": 66764.3,
  "tokens": 165,
  "ms_per_token": 404.632
}
```

### Building

Initialize the repository's submodule(s):

```shell
$ git submodule update --init --recursive
Submodule 'deps/cpp-httplib' (https://github.com/yhirose/cpp-httplib.git) registered for path 'deps/cpp-httplib'
Cloning into '/home/noname/repos/alpaca.http/deps/cpp-httplib'...
Submodule path 'deps/cpp-httplib': checked out 'a66a013ed78dee11701d6075c6b713307004a126'
```

Then simply follow the [Building from Source (MacOS/Linux)](#building-from-source-macoslinux) instructions below (has not been tested on Windows).

### Running

Enable the HTTP server by including the `-H` (or `--http`) command line option.

If no argument is given, will use the default hostname and port pair (currently `127.0.0.1` and `42000`). Otherwise, the argument must be a string of the form "hostname:port".

For example, to bind to `0.0.0.0` on port `12345`:

```shell
$ ./chat -m ./ggml-alpaca-7b-q4.bin -H 0.0.0.0:12345

```
When in HTTP mode (`-H`/`--http`), interactive mode is automatically disabled regardless of related any command line options.

---

# Alpaca.cpp

Run a fast ChatGPT-like model locally on your device. The screencast below is not sped up and running on an M2 Macbook Air with 4GB of weights. 


[![asciicast](screencast.gif)](https://asciinema.org/a/dfJ8QXZ4u978Ona59LPEldtKK)


This combines the [LLaMA foundation model](https://github.com/facebookresearch/llama) with an [open reproduction](https://github.com/tloen/alpaca-lora) of [Stanford Alpaca](https://github.com/tatsu-lab/stanford_alpaca) a fine-tuning of the base model to obey instructions (akin to the [RLHF](https://huggingface.co/blog/rlhf) used to train ChatGPT) and a set of modifications to [llama.cpp](https://github.com/ggerganov/llama.cpp) to add a chat interface. 

## Get Started (7B)

Download the zip file corresponding to your operating system from the [latest release](https://github.com/antimatter15/alpaca.cpp/releases/latest). On Windows, download `alpaca-win.zip`, on Mac (both Intel or ARM) download `alpaca-mac.zip`, and on Linux (x64) download `alpaca-linux.zip`. 

Download  [ggml-alpaca-7b-q4.bin](https://huggingface.co/Sosaka/Alpaca-native-4bit-ggml/blob/main/ggml-alpaca-7b-q4.bin) and place it in the same folder as the `chat` executable in the zip file. There are several options: 

Once you've downloaded the model weights and placed them into the same directory as the `chat` or `chat.exe` executable, run:

```
./chat
```

The weights are based on the published fine-tunes from `alpaca-lora`, converted back into a pytorch checkpoint with a [modified script](https://github.com/tloen/alpaca-lora/pull/19) and then quantized with llama.cpp the regular way. 

## Building from Source (MacOS/Linux)


```sh
git clone https://github.com/antimatter15/alpaca.cpp
cd alpaca.cpp

make chat
./chat
```


## Building from Source (Windows)

- Download and install CMake: <https://cmake.org/download/>
- Download and install `git`. If you've never used git before, consider a GUI client like <https://desktop.github.com/>
- Clone this repo using your git client of choice (for GitHub Desktop, go to File -> Clone repository -> From URL and paste `https://github.com/antimatter15/alpaca.cpp` in as the URL)
- Open a Windows Terminal inside the folder you cloned the repository to
- Run the following commands one by one:

```ps1
cmake .
cmake --build . --config Release
```

- Download the weights via any of the links in "Get started" above, and save the file as `ggml-alpaca-7b-q4.bin` in the main Alpaca directory.
- In the terminal window, run this command:
```ps1
.\Release\chat.exe
```
- (You can add other launch options like `--n 8` as preferred onto the same line)
- You can now type to the AI in the terminal and it will reply. Enjoy!

## Credit

This combines [Facebook's LLaMA](https://github.com/facebookresearch/llama), [Stanford Alpaca](https://crfm.stanford.edu/2023/03/13/alpaca.html), [alpaca-lora](https://github.com/tloen/alpaca-lora) and [corresponding weights](https://huggingface.co/tloen/alpaca-lora-7b/tree/main) by Eric Wang (which uses [Jason Phang's implementation of LLaMA](https://github.com/huggingface/transformers/pull/21955) on top of Hugging Face Transformers), and [llama.cpp](https://github.com/ggerganov/llama.cpp) by Georgi Gerganov. The chat implementation is based on Matvey Soloviev's [Interactive Mode](https://github.com/ggerganov/llama.cpp/pull/61) for llama.cpp. Inspired by [Simon Willison's](https://til.simonwillison.net/llms/llama-7b-m2) getting started guide for LLaMA. [Andy Matuschak](https://twitter.com/andy_matuschak/status/1636769182066053120)'s thread on adapting this to 13B, using fine tuning weights by [Sam Witteveen](https://huggingface.co/samwit/alpaca13B-lora). 


## Disclaimer

Note that the model weights are only to be used for research purposes, as they are derivative of LLaMA, and uses the published instruction data from the Stanford Alpaca project which is generated by OpenAI, which itself disallows the usage of its outputs to train competing models. 


