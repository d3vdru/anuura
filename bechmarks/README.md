# Inference and training benchmarks

List of models run on ada and turing with throughputs and other details documented.

| Hardware | Model | pp | tg | Code | Dev notes |
| :--- | :--- | :--- | :--- | :--- | :--- |
| ada 3x2080ti, 1x3080 | Qwen3-30B-A3B-Thinking-2507 Q6_K  | pp2048 2096.62 ± 289.67 | tg32 94.96 ± 0.91 | [scripts](https://github.com/d3vdru/anuura/blob/main/bechmarks/ada/run-job.sh) | @ojas.kataria: ada is on cuda 12.8, vllm has moved onto 13.x, hence using llama.cpp, benchmark done using [llama-benchy](https://github.com/eugr/llama-benchy) |
