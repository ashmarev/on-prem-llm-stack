# RTX2080Ti
## Цель исследования
Показать как эффективно использовать LLM в условиях ограниченной инфраструктуры (на примере 1 GPU). 
NVLink присутствует, но не используется в рамках данного теста.
## Вывод
Оптимальная нагрузка для одной 2080 Ti ≈ 8 concurrent запросов (572 tok/sec при 0,7s p99 TTFT). При дальнейшем увеличении concurrency наблюдается деградация: рост latency (p99 TTFT до 7.7s) и снижение throughput так как рост очереди KV-cache приводит к резкому увеличению TTFT.
В ходе тестирования было установлено, что при использовании квантования (awq_marlin) и типа данных float16 модель Qwen3-4B-AWQ эффективно размещается на GPU RTX 2080 Ti с доступным KV-cache около 5.8 GiB. С учётом фрагментации реальная стоимость токена составляет ~206 КБ, что даёт порядка 28K токенов в KV-cache.
Оптимальной конфигурацией является --max-num-seqs=8 и --max-num-batched-tokens=4096, при которой достигается баланс между throughput и latency. При увеличении concurrency система входит в saturation: KV-cache заполняется, очередь растёт, и TTFT резко увеличивается, что приводит к деградации производительности.
## Hardware
```
root@***-ai-server:/home/admin# lscpu
Architecture:                x86_64
  CPU op-mode(s):            32-bit, 64-bit
CPU(s):                      16
Vendor ID:                   GenuineIntel
  BIOS Vendor ID:            Intel(R) Corporation
  Model name:                Intel(R) Core(TM) i9-9900K CPU @ 3.60GHz
```
```
root@***-ai-server:/home/admin# free -h
               total        used        free      shared  buff/cache   available
Mem:            62Gi       6.6Gi       4.2Gi       103Mi        52Gi        56Gi
Swap:          8.0Gi       588Ki       8.0Gi
```
## OS
```
root@***-ai-server:/home/admin# hostnamectl
 Static hostname: ***-ai-server
       Icon name: computer-desktop
         Chassis: desktop 🖥️
      Machine ID: 8f06c2e61e1a4fa3b5ba688d90f41036
         Boot ID: b94845a8b30f4bada466e770d2beaa02
Operating System: Ubuntu 24.04.2 LTS              
          Kernel: Linux 6.17.0-20-generic
    Architecture: x86-64
 Hardware Vendor: Gigabyte Technology Co., Ltd.
  Hardware Model: Z390 GAMING SLI
Firmware Version: F10c
   Firmware Date: Wed 2019-12-18
    Firmware Age: 6y 3month 2w 4d 
```
## Общие программы
* apt-get install moreutils (для ts)
## GPU
### NVidia-smi
* sudo ubuntu-drivers autoinstall
* sudo reboot
* nvidia-smi
```  
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 580.126.09             Driver Version: 580.126.09     CUDA Version: 13.0     |
+-----------------------------------------+------------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
|                                         |                        |               MIG M. |
|=========================================+========================+======================|
|   0  NVIDIA GeForce RTX 2080 Ti     Off |   00000000:01:00.0  On |                  N/A |
| 63%   60C    P8             29W /  250W |      53MiB /  11264MiB |      0%      Default |
|                                         |                        |                  N/A |
+-----------------------------------------+------------------------+----------------------+
|   1  NVIDIA GeForce RTX 2080 Ti     Off |   00000000:02:00.0 Off |                  N/A |
| 34%   50C    P8             14W /  250W |       6MiB /  11264MiB |      0%      Default |
|                                         |                        |                  N/A |
+-----------------------------------------+------------------------+----------------------+

+-----------------------------------------------------------------------------------------+
| Processes:                                                                              |
|  GPU   GI   CI              PID   Type   Process name                        GPU Memory |
|        ID   ID                                                               Usage      |
|=========================================================================================|
|    0   N/A  N/A            2134      G   /usr/lib/xorg/Xorg                       33MiB |
|    0   N/A  N/A            2407      G   /usr/bin/gnome-shell                      9MiB |
|    1   N/A  N/A            2134      G   /usr/lib/xorg/Xorg                        4MiB |
+-----------------------------------------------------------------------------------------+
```
### Мониторинг GPU (только для серверных карт)
#### Игровые GPU
* nvidia-smi dmon | ts '%Y-%m-%d %H:%M:%S'
#### Серверные GPU
* wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2404/x86_64/cuda-keyring_1.1-1_all.deb
* sudo dpkg -i cuda-keyring_1.1-1_all.deb
* sudo apt-get update
* CUDA_VERSION=$(nvidia-smi | sed -E -n 's/.*CUDA Version: ([0-9]+)[.].*/\1/p')
sudo apt-get install --yes --install-recommends datacenter-gpu-manager-4-cuda${CUDA_VERSION}
* sudo apt-get install --yes --install-recommends datacenter-gpu-manager-4-cuda13
* sudo systemctl --now enable nvidia-dcgm
* dcgmi discovery -l
* dcgmi dmon -e 200,201,202,203,204,205 -d 1000 | ts '%Y-%m-%d %H:%M:%S'
* dcgmi dmon -e 1001,1002,1003 -d 1000 | ts '%Y-%m-%d %H:%M:%S'
### NVLink
* nvidia-smi topo -m
```
        GPU0    GPU1    CPU Affinity    NUMA Affinity   GPU NUMA ID
GPU0     X      NV2     0-15    0               N/A
GPU1    NV2      X      0-15    0               N/A

Legend:

  X    = Self
  SYS  = Connection traversing PCIe as well as the SMP interconnect between NUMA nodes (e.g., QPI/UPI)
  NODE = Connection traversing PCIe as well as the interconnect between PCIe Host Bridges within a NUMA node
  PHB  = Connection traversing PCIe as well as a PCIe Host Bridge (typically the CPU)
  PXB  = Connection traversing multiple PCIe bridges (without traversing the PCIe Host Bridge)
  PIX  = Connection traversing at most a single PCIe bridge
  NV#  = Connection traversing a bonded set of # NVLinks
```
## vLLM
* sudo apt-get install python3-venv
* python3 -m venv env
* source env/bin/activate
* pip install vllm==0.19.0
* touch start.sh
```
#!/bin/bash
export HF_HUB_OFFLINE=1
export CUDA_VISIBLE_DEVICES=1
export CUDA_DEVICE_ORDER=PCI_BUS_ID
/home/admin/vllm/env/bin/vllm serve Qwen/Qwen3-4B-AWQ \
--gpu-memory-utilization 0.90 \
--tensor-parallel-size 1 \
--max-num-seqs 8 \
--max-num-batched-tokens 4096 \
--dtype float16 \
--quantization awq_marlin \
--host 127.0.0.1 \
--port 8001                  
```  
* chmod +x start.sh
* source start.sh
* запрос к LLM:
```
curl -X POST http://127.0.0.1:8001/v1/chat/completions   -H "Content-Type: application/json"   -d '{
    "messages": [
      {
        "role": "user",
        "content": "Придумай диалог между учёным и художником о природе времени. Используй не менее 5 метафор из области физики и 3 — из искусства. Объём: 6 реплик. /no_think"
      }
    ],
    "model": "Qwen/Qwen3-4B-AWQ",
    "temperature": 0.5,
    "stream": false
  }'
```

* тест sweep:
```
for conc in 1 2 4 8 16 32; do
  echo "=== Concurrency $conc ==="
  vllm bench serve \
    --backend openai \
    --base-url http://127.0.0.1:8001 \
    --dataset-name random \
    --num-prompts 1024 \
    --random-input-len 400 \
    --random-output-len 150 \
    --request-rate inf \
    --max-concurrency $conc \
    --ignore-eos \
    --percentile-metrics ttft,itl,tpot,e2el \
    --metric-percentiles 50,90,95,99 \
    --save-result \
    --save-detailed \
    --result-filename result-conc-$conc.json
  sleep 10
done

```
* как просмотреть результат
```
printf "%-28s | %-11s | %-7s | %-12s | %-10s | %-10s\n" \
       "filename" "out_tok/s" "req/s" "p99 TTFT ms" "p99 ITL ms" "p99 TPOT ms"
echo "-----------------------------------------------------------------------"

for f in result-conc-*.json; do
  jq -r --arg fn "$f" '
    [
      $fn,
      (.output_throughput  | . * 100.0 | round / 100.0 | tostring),
      (.request_throughput | . * 100.0 | round / 100.0 | tostring),
      (.p99_ttft_ms        // "N/A" | . * 100.0 | round / 100.0 | tostring),
      (.p99_itl_ms         // "N/A" | . * 100.0 | round / 100.0 | tostring),
      (.p99_tpot_ms        // "N/A" | . * 100.0 | round / 100.0 | tostring)
    ] | @tsv
  ' "$f" | awk -F'\t' '{
      printf "%-28s | %-11s | %-7s | %-12s | %-10s | %-10s\n", \
             $1, $2, $3, $4, $5, $6
  }'
done
```
* результат
```
filename                     | out_tok/s   | req/s   | p99 TTFT ms  | p99 ITL ms | p99 TPOT ms
-----------------------------------------------------------------------
result-conc-1.json           | 112.66      | 0.75    | 102.6        | 10.14      | 8.74
result-conc-2.json           | 200.61      | 1.34    | 204.55       | 9.43       | 9.47      
result-conc-4.json           | 327.19      | 2.18    | 401.14       | 10.39      | 11.84
result-conc-8.json           | 572.37      | 3.82    | 712.77       | 12.82      | 14.29   
result-conc-16.json          | 530.98      | 3.54    | 3108.86      | 13.6       | 14.9      
result-conc-32.json          | 520.81      | 3.47    | 7781.66      | 20.55      | 15.09     
```
