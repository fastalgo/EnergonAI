<!-- <div  align="center">    
    <img src="https://user-images.githubusercontent.com/12018307/170214566-b611b131-fff1-41c0-9447-786a8a6f0bac.png" width = "600" height = "148" alt="Architecture" align=center />
</div> -->
# Energon-AI

![](https://img.shields.io/badge/Made%20with-ColossalAI-blueviolet?style=flat)
[![GitHub license](https://img.shields.io/github/license/hpcaitech/FastFold)](https://github.com/hpcaitech/ColossalAI-Inference/blob/main/LICENSE)


A Large-scale Model Inference System.
Energon-AI provides 3 levels of abstraction for enabling the large-scale model inference:
- **Runtime** - tensor parallel operations, pipeline parallel wrapper, distributed message queue, distributed checkpoint loading, customized CUDA kernels.
- **Engine** - encapsulate the single instance multiple devices (SIMD) execution with the remote procedure call, which acts as the single instance single device (SISD) execution.
- **Serving** - batching requests, managing engines.

For models trained by [Colossal-AI](https://github.com/hpcaitech/ColossalAI), they can be seamlessly transferred to Energon-AI.
For single-device models, they require manual coding works to introduce tensor parallelism and pipeline parallelism.

At present, we pre-build distributed Bert, GPT, and ViT models.  
For GPT, it extends to at most 175B parameters, which is called [GPT3](https://arxiv.org/abs/2005.14165).  
For Bert, Google reports a [super-large Bert with 481B parameters](https://mlcommons.org/en/training-normal-11/) in MLPerf-Training v1.1 open, indicating that Bert can also extend to large-scale.

### Installation
``` bash
$ git clone git@github.com:hpcaitech/EnergonAI.git
$ pip install -r requirements.txt
$ pip install .
```

### Huggingface GPT2 Generation Task Case

``` bash
# Download checkpoint
$ wget https://huggingface.co/gpt2/blob/main/pytorch_model.bin
# Download files for tokenizer
$ wget https://huggingface.co/gpt2/blob/main/tokenizer.json
$ wget https://huggingface.co/gpt2/blob/main/vocab.json
$ wget https://huggingface.co/gpt2/blob/main/merges.txt

# Launch the service
export PYTHONPATH=~/ColossalAI-Inference/examples/hf_gpt2
energonai service init --config_file=~/ColossalAI-Inference/hf_gpt2/hf_gpt2_config.py

# Request for the service
Method 1: 
    FastAPI provides an automatic API docs, you can forward 
    http://127.0.0.1:8005/docs and make request with the graphical interface.
Method 2:
    curl -X 'GET' \
    'http://127.0.0.1:8005/run_hf_gpt2/I%20do%20not?max_seq_length=16' \
    -H 'accept: application/json' 
```

### Large-scale Model Inference Performance
#### Scaling Ability

Here GPT3-12-layers in FP16 is adopted.  
Here a node with 8 A100 80 GB GPUs is adopted. GPUs are fully connected with NvLink.   
Energon-AI adopts the redundant computation elimination method. The method is first raised in [EffectiveTransformer](https://github.com/bytedance/effective_transformer), and our implementation refers to [TurboTransformer](https://github.com/Tencent/TurboTransformers/blob/master/turbo_transformers/layers/kernels/gpu_transpose_kernel.cu).  
Here the sequence length is set the half of the padding length.
<div  align="center">    
    <img src="https://user-images.githubusercontent.com/12018307/168971637-ffd1d6ba-44bb-4043-a275-3dc2a008c048.png" width = "600" height = "240" alt="Architecture" align=center />
</div>

#### Latency
Here GPT3 in FP16 is adopted.  
Here a node with 8 A100 80 GB GPUs is adopted. Every two GPUs are connected with NvLink.  
Here the sequence length is set the half of the padding length when using redundant computation elimination method, which is the Energon-AI(RM).  
Here FasterTransformer is adopted in comparison and it does not support the redundant computation elimination method in the distributed execution.
<div  align="center">    
    <img src="https://user-images.githubusercontent.com/12018307/169728315-8ac95e4f-3e81-44e5-b82b-5873ffe85351.png" width = "600" height = "300" alt="Architecture" align=center />
</div>

#### Batching
Energon-AI dynamically selects the batch processing with the highest priority regarding the waiting time, batch size, batch expansion possibility (based on the sentence length after padding).
Our dynamic batching method is inspired by the DP algorithm from [TurboTransformer](https://dl.acm.org/doi/10.1145/3437801.3441578).  
Here FIFO batching is selected in comparison.
<div  align="center">    
    <img src="https://user-images.githubusercontent.com/12018307/170616782-18fae36f-75cd-4e7b-bc0b-c8998be1e540.png" width = "400" height = "100" alt="Architecture" align=center />
</div>


### Contributing

If interested in making your own contribution to the project, please refer to [Contributing](./CONTRIBUTING.md) for guidance.

Thanks so much!

### Technical Overview

<div  align="center">    
    <img src="https://user-images.githubusercontent.com/12018307/168971629-6df3232b-85a7-43ce-95df-f067e7e5959c.png" width = "480" height = "500" alt="Architecture" align=center />
</div>

<!-- 

![image (1)](https://user-images.githubusercontent.com/12018307/168971641-aebe986a-7e9d-4c66-9ced-4e8b7a1628e2.png)
![batch drawio](https://user-images.githubusercontent.com/12018307/168971644-35393802-7d8b-4e13-9428-340f7328616c.png) -->

