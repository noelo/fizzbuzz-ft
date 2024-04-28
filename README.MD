# Fine Tuning using axolotl

[WARNING] Work In Progress

* Using Runpod.io fire up a training pod with a 4090 GPU.

* Once ready, enter the pod

```
cd /workspace/axolotl
git clone https://github.com/OpenAccess-AI-Collective/axolotl
cd axolotl
pip3 install -e .[flash-attn]
pip3 install -U git+https://github.com/huggingface/peft.git

cd /tmp && git clone https://github.com/noelo/fizzbuzz-ft && cd -

export WANDB_API_KEY=.....
huggingface-cli login --token ..... --add-to-git-credential
cd -
```

## Fine Tune the Model & Merge the LoRA weights

```
accelerate launch scripts/finetune.py /tmp/fizzbuzz-ft/config.yml

python3 -m axolotl.cli.merge_lora /tmp/fizzbuzz-ft/config.yml 
```

OR

```
wget https://gist.githubusercontent.com/mlabonne/a3542b0519708b8871d0703c938bba9f/raw/60abc5afc07f9d843bc23d56f4e0b7ab072c4a62/merge_peft.py

python merge_peft.py --base_model=codellama/CodeLlama-7b-hf --peft_model=./lora-out --hub_id=loracamel
```

## Quantize to GGUF
```
pip install -r llama.cpp/requirements.txt
sudo dnf install git-lfs
mkdir loracamel-model
cd loracamel-model/
git clone https://huggingface.co/noeloco/loracamel
python ../llama.cpp/convert.py loracamel --outtype f16 --outfile loca-camel.fp16
../llama.cpp/quantize loca-camel.fp16 q4_k_m loca-camel.fp16.q4_k_m
../llama.cpp/quantize loca-camel.fp16 "q4_k_m" loca-camel.fp16.q4_k_m
```