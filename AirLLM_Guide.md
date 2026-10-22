# AirLLM: Hardware Requirements, Installation, Setup, and Usage

> **Purpose:** A practical guide to running large language models (LLMs)
> with AirLLM, including hardware planning, installation, first
> inference, model storage, troubleshooting, and safe usage.
>
> **Checked:** 22 October 2026\
> **Project:** https://github.com/lyogavin/airllm\
> **Package:** https://pypi.org/project/airllm/

## 1. What is AirLLM?

AirLLM is a Python library for running supported Hugging Face language
models by streaming model layers from storage/system memory rather than
keeping the entire model in GPU memory at once. This can make inference
possible on hardware with limited VRAM.

Important trade-off: **lower VRAM does not mean low total hardware
requirements or high speed**. Large models still need substantial disk
space, system RAM, storage bandwidth, and patience. AirLLM is generally
most useful when a model otherwise does not fit in GPU memory and slower
inference is acceptable.

AirLLM is primarily an **inference** approach. Some newer releases also
include specific streamed fine-tuning examples, but training support is
model- and version-specific; do not assume every model can be fine-tuned
this way.

## 2. Hardware requirements

Requirements vary by model architecture, parameter count,
precision/quantization, context length, batch size, AirLLM version, and
operating system. Treat the figures below as planning guidance, not
guarantees.

### 2.1 Practical workstation guidance

  ------------------------------------------------------------------------------
  Goal                 GPU / VRAM      System RAM    Free storage Notes
                         planning                        planning 
  ------------ ------------------ --------------- --------------- --------------
  Experiment           Integrated        8--16 GB       20--50 GB Best starting
  with a small   graphics or 4--6                                 point;
  1B--3B model   GB VRAM may work                                 CPU-only may
                   for compatible                                 work but can
                           models                                 be slow

  Run a 7B--8B   4--8 GB VRAM can       16--32 GB       30--80 GB A GPU is
  model        be enough for some                                 helpful; leave
                   configurations                                 room for model
                                                                  shards and
                                                                  cache

  Run a 14B     6--12 GB VRAM may           32 GB      50--120 GB Depends
  model                  work for     recommended                 strongly on
                compatible models                                 model format
                                                                  and context
                                                                  length

  Experiment    AirLLM may reduce       32--64 GB    100--200+ GB Expect slow
  with                 VRAM needs     recommended                 generation and
  30B--34B          substantially                                 significant
  models                                                          disk I/O

  Experiment       AirLLM reports  64 GB RAM is a    150--250+ GB Not a promise
  with 70B                   some       practical                 for every 70B
  models           configurations          target                 model; model
                     running with                                 loading and
                 around 4 GB VRAM                                 conversion
                                                                  need space

  Very large      Model-specific; 64--128+ GB may  Hundreds of GB Check the
  models       published examples       be useful        to 1 TB+ exact model
  (100B+)                 may use                                 instructions
                  single-digit or                                 and disk
                 low-double-digit                                 footprint
                          GB VRAM                                 before
                                                                  downloading
  ------------------------------------------------------------------------------

**Why storage estimates are ranges:** downloaded checkpoint size depends
on parameter count, data type, and quantization. AirLLM may also create
split/layer-shard files, so temporary free space can be much larger than
the final model size. Keep additional space available for the Hugging
Face cache and generated files.

### 2.2 What matters most?

1.  **GPU VRAM:** AirLLM streams layers, but the active layer,
    attention/KV cache, temporary tensors, and runtime overhead still
    need memory.
2.  **System RAM:** Needed for Python, model metadata, CPU-side tensors,
    caching, and other processes. More RAM is especially useful for very
    large models.
3.  **SSD capacity and speed:** NVMe SSD is strongly preferred. Layer
    streaming can involve repeated disk reads; a slow HDD can make
    inference painfully slow.
4.  **GPU compute:** A supported NVIDIA GPU with a compatible
    PyTorch/CUDA setup is a common path. A powerful GPU may improve
    throughput, but AirLLM's storage and streaming overhead can remain
    significant.
5.  **CPU and cooling:** A modern multi-core CPU and adequate cooling
    help with preprocessing and host-side work.
6.  **Operating system:** Linux is a common target. Windows with a
    compatible PyTorch/CUDA environment may work. On macOS, the project
    documents Apple Silicon support and requires the relevant
    MLX/PyTorch setup for that path.

### 2.3 Does AirLLM make every huge model practical?

No. Claims such as "70B on 4 GB VRAM" describe particular tested
configurations, not a universal minimum. Compatibility, RAM, disk space,
prompt length, model architecture, and speed all matter.
Mixture-of-Experts (MoE) models and dense models can have very different
memory and storage behavior.

For a dependable workstation, prioritize **RAM, fast SSD capacity, and a
supported model** rather than buying a GPU based only on a minimum-VRAM
claim.

## 3. Software prerequisites

Recommended baseline:

-   64-bit Python in a virtual environment. Use a Python version
    supported by the installed AirLLM and PyTorch releases.
-   A compatible PyTorch build.
-   NVIDIA driver and CUDA-compatible PyTorch if using an NVIDIA GPU.
-   Git (optional; useful for examples and source installs).
-   Hugging Face account/token only if the chosen model is gated or
    requires authentication.
-   Enough free disk space for the model, its cache, and
    AirLLM-generated shards.

Check your environment:

### Windows PowerShell

``` powershell
python --version
python -m pip --version
nvidia-smi
```

`nvidia-smi` is only expected to work if an NVIDIA driver/GPU is
installed. If the command is not found, verify the driver and PATH.

### Linux

``` bash
python3 --version
python3 -m pip --version
nvidia-smi
df -h
free -h
```

### macOS

``` bash
python3 --version
python3 -m pip --version
system_profiler SPHardwareDataType
```

AirLLM's macOS path is intended for Apple Silicon; check the current
project README for the exact dependency requirements.

## 4. Installation

### 4.1 Create a virtual environment

**Windows PowerShell**

``` powershell
mkdir airllm-demo
cd airllm-demo
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
```

If PowerShell blocks activation, either use a permitted execution policy
for your user or run the environment's Python directly:

``` powershell
.\.venv\Scripts\python.exe -m pip install --upgrade pip
```

**Linux**

``` bash
mkdir -p airllm-demo
cd airllm-demo
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
```

### 4.2 Install PyTorch

Install a PyTorch build compatible with your operating system, Python
version, and GPU driver by following the official selector:

https://pytorch.org/get-started/locally/

For a CPU-only trial, use the CPU installation command provided by the
PyTorch selector. For an NVIDIA GPU, use the matching CUDA build; do not
guess a CUDA wheel from an unrelated tutorial.

Verify PyTorch:

``` bash
python -c "import torch; print('PyTorch:', torch.__version__); print('CUDA available:', torch.cuda.is_available()); print('GPU:', torch.cuda.get_device_name(0) if torch.cuda.is_available() else 'CPU')"
```

A `False` CUDA result means PyTorch cannot currently access CUDA.
Resolve that before expecting GPU inference.

### 4.3 Install AirLLM

``` bash
python -m pip install --upgrade airllm
```

Check installation:

``` bash
python -c "import airllm; print('AirLLM import successful')"
python -m pip show airllm torch transformers
python -m pip check
```

For reproducible projects, record the working versions after testing:

``` bash
python -m pip freeze > requirements-lock.txt
```

AirLLM and Transformers evolve quickly. If an existing model fails after
an upgrade, create a clean virtual environment and follow the
compatibility notes for that model rather than repeatedly changing
packages in a production environment.

## 5. First inference: a small model

Start with a model you have confirmed is supported and small enough for
your available RAM/storage. The example below uses a Hugging Face model
ID; availability and compatibility may change.

Create `run_airllm.py`:

``` python
from airllm import AutoModel

MODEL_ID = "Qwen/Qwen2.5-3B-Instruct"
MAX_LENGTH = 128

def main() -> None:
    # First run downloads the model and may prepare AirLLM layer shards.
    model = AutoModel.from_pretrained(MODEL_ID)

    prompts = [
        "Explain AirLLM in simple terms."
    ]

    tokens = model.tokenizer(
        prompts,
        return_tensors="pt",
        return_attention_mask=False,
        truncation=True,
        max_length=MAX_LENGTH,
        padding=True,
    )

    # The project's examples commonly send token IDs to CUDA.
    # This assumes a working CUDA-enabled PyTorch installation.
    input_ids = tokens["input_ids"].cuda()

    output = model.generate(
        input_ids,
        max_new_tokens=128,
        do_sample=False,
        use_cache=True,
    )

    print(model.tokenizer.decode(output[0], skip_special_tokens=True))

if __name__ == "__main__":
    main()
```

Run it:

``` bash
python run_airllm.py
```

**Important:** The `.cuda()` line assumes CUDA is available. If you are
on CPU-only hardware or the selected AirLLM/model version documents a
different device path, adapt the example to that supported path. AirLLM
has had CPU support, but CPU performance can be slow and exact behavior
depends on the version and model. Check the current README before
changing device placement.

If the model is gated, accept its license on Hugging Face and
authenticate as described in Section 8.

## 6. Use a different model

AirLLM's generic entry point is:

``` python
from airllm import AutoModel

model = AutoModel.from_pretrained("ORGANIZATION/MODEL_NAME")
```

Replace the model ID with the exact Hugging Face repository ID or a
supported local model path.

Before choosing a model:

1.  Read its model card and license.
2.  Check whether it is gated and whether you need to log in.
3.  Confirm the model architecture and Transformers version are
    supported by your AirLLM version.
4.  Check the full download size and leave extra space for generated
    shards.
5.  Start with a short prompt and small `max_new_tokens` value.
6.  Check the current AirLLM README for model-specific exceptions and
    options.

The project lists support across many families, including Llama, Qwen,
DeepSeek, Mistral/Mixtral, Phi, Gemma, ChatGLM, Baichuan, InternLM, and
Yi. That does **not** guarantee every checkpoint or every new model
variant will work without adjustments.

## 7. Model downloads, caches, and disk management

The first load may download many gigabytes and prepare layer shards. It
can take a long time and may require more temporary space than expected.

Practical recommendations:

-   Use an SSD with enough free space before starting.
-   Avoid interrupting the first download/shard preparation unless
    necessary.
-   Identify where Hugging Face caches files on your system; the default
    is commonly under the user's cache directory.
-   If the project supports `layer_shards_saving_path` for your chosen
    version, set it to a drive with sufficient free space.
-   Do not delete original model files until you have verified the
    generated shards work and you understand whether the originals are
    needed again.
-   Avoid storing duplicate copies of the same large checkpoint unless
    necessary.

Example with a shard path (confirm this parameter against your installed
version):

``` python
from airllm import AutoModel

model = AutoModel.from_pretrained(
    "Qwen/Qwen2.5-3B-Instruct",
    layer_shards_saving_path="./airllm-shards",
)
```

If an argument is rejected by your installed version, consult the
current README and the installed package's signature rather than
assuming an example from another release still applies.

## 8. Hugging Face authentication for gated models

Some model repositories require license acceptance or authentication.
Use the official Hugging Face instructions:

https://huggingface.co/docs/huggingface_hub/quick-start

Typical login command:

``` bash
hf auth login
```

If that command is unavailable in your installed Hugging Face Hub
version, follow its current documentation for the appropriate CLI
command.

Security notes:

-   Never commit an access token to Git.
-   Do not paste tokens into source files, screenshots, or public logs.
-   Use environment variables or the supported credential store.
-   A login token does not bypass a model's license or access approval.

## 9. Generation settings

The most relevant controls in a basic example are:

  -----------------------------------------------------------------------
  Parameter               Purpose                 Practical advice
  ----------------------- ----------------------- -----------------------
  `max_new_tokens`        Maximum number of       Start with 64--128 to
                          generated tokens        test; longer outputs
                                                  take more time

  `do_sample`             Enables sampling rather `False` is useful for
                          than deterministic      repeatable smoke tests
                          decoding                

  `temperature`           Controls sampling       Use only when
                          randomness when         `do_sample=True`;
                          sampling is enabled     supported behavior
                                                  depends on the model

  `top_p`                 Nucleus sampling        Tune only when sampling
                          threshold               

  `max_length` in         Maximum input token     Truncation can discard
  tokenization            length in the example   important prompt
                                                  content

  Batch size              Number of prompts       Start with one prompt;
                          processed together      larger batches may use
                                                  more memory

  Context length          Input plus generated    Longer context can
                          sequence length         increase memory use and
                          supported by the model  latency
  -----------------------------------------------------------------------

For chat/instruct models, use the model's recommended chat template if
required. Plain text prompts can work but may produce lower-quality or
oddly formatted responses for some instruction-tuned models.

## 10. Run as a local Python utility

A minimal reusable function:

``` python
from airllm import AutoModel

MODEL_ID = "Qwen/Qwen2.5-3B-Instruct"
MAX_INPUT_TOKENS = 256

def load_model():
    return AutoModel.from_pretrained(MODEL_ID)

def generate_text(model, prompt: str, max_new_tokens: int = 128) -> str:
    tokens = model.tokenizer(
        [prompt],
        return_tensors="pt",
        return_attention_mask=False,
        truncation=True,
        max_length=MAX_INPUT_TOKENS,
        padding=True,
    )

    input_ids = tokens["input_ids"].cuda()
    output = model.generate(
        input_ids,
        max_new_tokens=max_new_tokens,
        do_sample=False,
        use_cache=True,
    )
    return model.tokenizer.decode(output[0], skip_special_tokens=True)

if __name__ == "__main__":
    llm = load_model()
    print(generate_text(llm, "Give me three uses of local LLMs."))
```

This sample uses CUDA and is not a universal CPU/macOS wrapper. Confirm
the device behavior for your installed AirLLM release before reusing it
on another platform.

## 11. Use from FastAPI (basic integration pattern)

For an API service, load the model once at application startup rather
than reloading it for every request. The following is a **minimal
demonstration**, not a production-ready service; model loading may take
a long time, inference may block, and concurrency must be controlled.

Install FastAPI and Uvicorn in the same virtual environment:

``` bash
python -m pip install fastapi uvicorn
```

Create `api.py`:

``` python
from contextlib import asynccontextmanager
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel, Field
from airllm import AutoModel

MODEL_ID = "Qwen/Qwen2.5-3B-Instruct"
llm = None

@asynccontextmanager
async def lifespan(app: FastAPI):
    global llm
    llm = AutoModel.from_pretrained(MODEL_ID)
    yield
    llm = None

app = FastAPI(title="AirLLM Local API", lifespan=lifespan)

class GenerateRequest(BaseModel):
    prompt: str = Field(min_length=1, max_length=10000)
    max_new_tokens: int = Field(default=128, ge=1, le=512)

@app.get("/health")
def health():
    return {"status": "ok", "model_loaded": llm is not None}

@app.post("/generate")
def generate(request: GenerateRequest):
    if llm is None:
        raise HTTPException(status_code=503, detail="Model is not loaded")

    try:
        tokens = llm.tokenizer(
            [request.prompt],
            return_tensors="pt",
            return_attention_mask=False,
            truncation=True,
            max_length=256,
            padding=True,
        )
        input_ids = tokens["input_ids"].cuda()
        output = llm.generate(
            input_ids,
            max_new_tokens=request.max_new_tokens,
            do_sample=False,
            use_cache=True,
        )
        text = llm.tokenizer.decode(output[0], skip_special_tokens=True)
        return {"text": text}
    except Exception as exc:
        # Log full details server-side in a real service; avoid exposing internals.
        raise HTTPException(status_code=500, detail="Inference failed") from exc
```

Run:

``` bash
uvicorn api:app --host 127.0.0.1 --port 8000
```

Open `http://127.0.0.1:8000/docs`.

Before exposing an API beyond localhost, add authentication,
request-size limits, rate limiting, logging, timeouts, and a queue or
lock so concurrent requests do not exhaust memory or interfere with a
single model instance. Do not run multiple worker processes unless you
intentionally want each worker to load its own model copy. Replace
`.cuda()` with the device path supported by your setup.

## 12. Troubleshooting

### `torch.cuda.is_available()` returns `False`

-   Confirm an NVIDIA GPU and driver are installed.
-   Install the PyTorch build matching your environment using the
    official PyTorch selector.
-   Verify you are using the intended virtual environment.
-   Restart the terminal after driver changes and rerun the check.

### Out of memory (GPU or system RAM)

-   Reduce batch size, input length, and `max_new_tokens`.
-   Close other GPU/RAM-heavy processes.
-   Try a smaller model or a compatible lower-memory checkpoint.
-   Check that the architecture/model variant is supported.
-   Remember that layer streaming reduces memory pressure but does not
    eliminate temporary allocations or KV-cache requirements.

### No space left on disk

-   Check free disk space before retrying.
-   Review Hugging Face cache and AirLLM shard directories.
-   Keep a backup of any model files you need before deleting caches.
-   Ensure the drive has room for both downloaded weights and prepared
    shards.

### First model load is extremely slow

-   Large models require large downloads and may need shard preparation.
-   Use an SSD, a stable internet connection, and a model appropriate
    for your storage.
-   Wait for the initial preparation to complete before judging
    steady-state behavior.

### Transformers / model architecture error

-   Read the AirLLM README and the model card.
-   Check the installed versions of `airllm`, `transformers`, `torch`,
    and `accelerate`.
-   Use a clean virtual environment to test compatible versions.
-   Some newly released or specialized models require a particular
    Transformers build or extra packages.

### Import or dependency conflicts

``` bash
python -m pip check
python -m pip show airllm transformers torch accelerate
```

If necessary, recreate the virtual environment and install PyTorch
first, then AirLLM. Avoid mixing packages installed globally with
packages installed inside the environment.

### Generated output is poor or repeats the prompt

-   Check the model's recommended prompt/chat template.
-   Ensure the correct instruct/chat checkpoint is selected.
-   Verify tokenizer settings and context limits.
-   Try a short, clear prompt and deterministic generation first.

## 13. Performance expectations and alternatives

AirLLM trades memory footprint for additional streaming and storage
work. It is a useful option when a large model otherwise cannot fit, but
it may be slower than serving a model fully resident in VRAM.

Consider alternatives when your hardware allows them:

-   **Ollama:** convenient local model management and API.
-   **llama.cpp:** broad CPU/GPU support and quantized GGUF models.
-   **Transformers + bitsandbytes:** quantized inference on compatible
    hardware.
-   **vLLM:** high-throughput serving on supported GPU configurations.
-   **MLX:** Apple Silicon-oriented model workflows.

Choose based on model compatibility, license, latency, throughput,
platform, and RAM/VRAM---not only the largest parameter count you can
load.

## 14. Security, privacy, and licensing

-   Local inference can keep prompts on your machine after model files
    are downloaded, but downloading models still contacts external
    services.
-   Review model license terms, acceptable-use policies, and any
    restrictions before using a model commercially.
-   Treat downloaded model repositories and remote-code requirements as
    untrusted until reviewed.
-   Do not expose an unauthenticated generation API to the public
    internet.
-   Keep API tokens and credentials out of source control.
-   For sensitive data, check logs, caches, crash dumps, and telemetry
    settings as well as the inference process itself.

## 15. Suggested installation checklist

-   [ ] Confirm OS, Python version, CPU, RAM, GPU/VRAM, and free SSD
    space.
-   [ ] Pick a small, supported model for the first test.
-   [ ] Create and activate a dedicated virtual environment.
-   [ ] Install a compatible PyTorch build.
-   [ ] Verify `torch.cuda.is_available()` if using NVIDIA CUDA.
-   [ ] Install AirLLM and run `pip check`.
-   [ ] Confirm model license and Hugging Face access requirements.
-   [ ] Download/load the model and complete a short generation test.
-   [ ] Record tested package versions and model ID.
-   [ ] Only then integrate it into a notebook, script, or API.

## 16. Official references

-   AirLLM GitHub: https://github.com/lyogavin/airllm
-   AirLLM on PyPI: https://pypi.org/project/airllm/
-   PyTorch installation selector:
    https://pytorch.org/get-started/locally/
-   Hugging Face Hub quick start:
    https://huggingface.co/docs/huggingface_hub/quick-start
-   Hugging Face model hub: https://huggingface.co/models

**Version note:** AirLLM, PyTorch, Transformers, model IDs, and their
compatibility requirements change frequently. Before a fresh
installation, compare these instructions with the current official
AirLLM README and the selected model card.
