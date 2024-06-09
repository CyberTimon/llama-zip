# llama-zip

`llama-zip` is a command-line utility for lossless text compression and decompression. It functions by leveraging a user-provided LLM (large language model) as the probabilistic model for an [arithmetic coder](https://en.wikipedia.org/wiki/Arithmetic_coding). This allows `llama-zip` to achieve high compression ratios for structured or natural language text, as fewer bits are needed to encode tokens that the LLM predicts with high confidence. By employing a sliding context window, `llama-zip` is not limited by the LLM's maximum context length and can handle arbitrarily long input text. The main limitation of `llama-zip` is that the speed of compression and decompression is limited by the LLM's inference speed.

![Interactive Mode Demo: Lorem Ipsum Text](lorem_ipsum_demo.gif)

## Compression Ratio

The compression ratio that `llama-zip` can achieve on a particular input depends on which LLM is used as well as the window overlap parameter (which determines how much of the context window is retained when sliding). The following table shows some compression ratios that `llama-zip` can achieve when using the [Llama 3 8B (Q4_K_M)](https://huggingface.co/QuantFactory/Meta-Llama-3-8B-GGUF) model with a window overlap of 25%. The files tested include all the text files from the standard [Calgary Corpus](http://www.data-compression.info/Corpora/CalgaryCorpus/), as well as `llama-zip`'s own source code. The compression ratios are calculated as the ratio of the original file size to the compressed file size, so higher values indicate better compression.

| File         | Ratio |
| ------------ | ----- |
| bib          | 8.523 |
| book1        | 6.943 |
| book2        | 8.127 |
| news         | 5.590 |
| paper1       | 7.637 |
| paper2       | 8.375 |
| progc        | 4.425 |
| progl        | 5.194 |
| progp        | 6.309 |
| trans        | 9.810 |
| llama_zip.py | 5.859 |

## Setup

```sh
git clone https://github.com/alexbuz/llama-zip.git
cd llama-zip
pip3 install .
```

### LLM Download

To use `llama-zip`, you must first download an LLM that is compatible with [llama.cpp](https://github.com/ggerganov/llama.cpp), such as [Llama 3 8B](https://huggingface.co/QuantFactory/Meta-Llama-3-8B-GGUF). Make sure to download a quantized version (one of the `.gguf` files listed on the "Files and versions" tab on Hugging Face) that is small enough to fit in your system's memory.

## Usage

```
llama-zip <llm_path> [options] <mode> [input]
```

### Modes

`llama-zip` supports three modes of operation:

1. **Compress mode** (specified by the `-c` or `--compress` flag): The string to be compressed can be provided as an argument or piped to stdin. The compressed output will be encoded in base64 and printed to stdout.
2. **Decompress mode** (specified by the `-d` or `--decompress` flag): The compressed string can be provided as an argument or piped to stdin. The decompressed output will be printed to stdout.
3. **Interactive mode** (specified by the `-i` or `--interactive` flag): A prompt is displayed where the user can enter strings to be compressed or decompressed. When a base64-encoded string is entered, it will be decompressed; otherwise, the entered string will be compressed. After each compression or decompression operation, the user is prompted to enter another string. To exit interactive mode, press `Ctrl+C`.
    - **Note:** If you would like to compress a string that consists entirely of base64 characters (i.e., letters, numbers, `+`, and `/`, without any other symbols or spaces), you must use compression mode directly, as interactive mode assumes that base64-encoded strings are meant to be decompressed and will result in nonsensical output if the input did not come from a compression operation. Alternatively, you can add a non-base64 character to your string (such as a space at the end) if you don't mind your string being compressed with that extra character.

### Options

- `-w`, `--window-overlap`: The number of tokens to overlap between the end of the previous context window and the start of the next window, when compressing a string whose length exceeds the LLM's maximum context length. This can be specified as a percentage of the LLM's context length or as a fixed number of tokens. The default is `0%`, meaning that the context window is cleared entirely when it is filled. Higher values can improve compression ratios but will slow down compression and decompression, since parts of the text will need to be re-evaluated when the context window slides. Note that when decompressing, the window overlap must be set to the same value that was used during compression in order to recover the original text.
- `--n_gpu_layers`: The `--n_gpu_layers` argument in the code specifies the number of layers in the model that should be offloaded to the GPU for computation. This can significantly speed up the processing time, especially for larger models, as the GPU is typically much faster at performing matrix operations than a CPU. If `--n_gpu_layers` is set to -1 or None, all layers of the model will be offloaded to the GPU. Check [llama.cpp's](https://github.com/ggerganov/llama.cpp) readme for better understanding of this parameter.

### Examples

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1piHwN-9imkyQkXoX4NMrf8-WeebeZKgj?usp=sharing)

#### Compression
- Compressing a string:
    ```sh
    llama-zip /path/to/Meta-Llama-3-8B.Q8_0.gguf -c "The quick brown fox jumps over the lazy dog."
    # Output: SxapgbY
    ```

- Compressing text from a file:
    ```sh
    llama-zip /path/to/Meta-Llama-3-8B.Q8_0.gguf -c < /path/to/gettysburg_address.txt
    # Output: 4vTMmKKTXWAcNZwPwkqN84
    ```

- Compressing text from a file and saving the output to another file:
    ```sh
    llama-zip /path/to/Meta-Llama-3-8B.Q8_0.gguf -c < /path/to/input.txt > /path/to/output.compressed
    ```

#### Decompression
- Decompressing a compressed string:
    ```sh
    llama-zip /path/to/Meta-Llama-3-8B.Q8_0.gguf -d SxapgbY
    # Output: The quick brown fox jumps over the lazy dog.
    ```

- Decompressing text from a file:
    ```sh
    llama-zip /path/to/Meta-Llama-3-8B.Q8_0.gguf -d < /path/to/input.compressed
    # Output: [decompressed text]
    ```

- Decompressing text from a file and saving the output to another file:
    ```sh
    llama-zip /path/to/Meta-Llama-3-8B.Q8_0.gguf -d < /path/to/input.compressed > /path/to/output.txt
    ```
