# indic-redteaming

We red-teamed Cohere's Tiny Aya Global in English and Telugu to see if its refusals still hold when the same harmful request is asked in Telugu. They mostly don't, so we also looked inside the model to see why.

This was built for the Lexsi AI Safety and Alignment Hackathon (Track 2a, problems 2 and 4).

## Why

We came across a research paper which showed that LLM safeguards are a lot easier to break when you approach the model with malicious prompts in languages it wasn't properly trained on, or rather security tested on. These are mostly native, less known languages. People often use these LLMs in their native language, so if a malicious person asks the AI to do something it wouldn't do in English, and it works, that's a breach of security.

## What we found

We took 30 behaviors from JailbreakBench (3 per category), each with a harmless twin, and asked Tiny Aya Global all of them in English and Telugu, plain and wrapped in 5 attacks. That gave 410 replies, which we labeled by hand.

In English it complied with 16.7% of the harmful prompts. In Telugu it was 35.4% (chi-square p = 0.0001). On the exact same prompt and attack, 44 were safe in English but harmful in Telugu, and only 11 went the other way.

Most of the gap comes from fiction framings. With the villain framing it went from 21% in English to 67% in Telugu. It's also worst where it matters most: on high-severity prompts it was 6% in English and 31% in Telugu.

| Attack | English | Telugu |
|---|---|---|
| villain | 21% | 67% |
| fictional_world | 17% | 47% |
| story | 13% | 43% |
| screenplay | 27% | 30% |
| plain | 7% | 13% |
| pair | 16% | 8% |

To see why, we used SafeTune to pull out the model's refusal direction (layer 21), using English prompts that aren't in the test set. If English harmless prompts are 0 and English harmful ones are 1, Telugu harmful prompts only reach 0.80 (p = 0.019). The villain framing pushes it down to 0.11 in English and 0.01 in Telugu. This signal predicts whether the model refuses or complies pretty well (AUC 0.94 in English, 0.93 in Telugu), so the gap really does trace back to the model reacting less strongly to Telugu.

We also ran AuditKIT on the replies. It got the same compliance numbers, but its multilingual refusal detector didn't catch a single Telugu refusal (0%, vs 44% when we read them), so it can't score Telugu yet.

Charts and tables are in `results/`.

## Notebooks

| Notebook | What it does | Needs |
|---|---|---|
| `00_exploration` | Early checks: romanized Telugu, Global vs Fire, a first pilot | 2 GPUs |
| `01_build_prompts` | Picks the JBB behaviors and adds PAIR prompts | CPU |
| `02_generate` | Runs Tiny Aya Global on everything | GPU |
| `03_analyze` | Turns our labels into the numbers and charts | CPU |
| `04_lexsi_curatorkit_auditkit` | Cleans the prompts with CuratorKIT, scores with AuditKIT | CPU |
| `05_safetune_interpret` | Refusal direction with SafeTune | GPU |
| `06_publish_benchmark` | Writes the dataset card and uploads it | CPU |

Translation (between 01 and 02) and labeling (between 02 and 03) were done in Google Sheets. The formulas and column names are written in those notebooks.

## Running it yourself

1. Get a Hugging Face token and accept the Tiny Aya license on its model page.
2. On Kaggle, add the token as a secret called `HF_TOKEN`. Anywhere else, set it as an environment variable.
3. `pip install -r requirements.txt` (SafeTune needs Python 3.12 or newer).
4. If you want our exact numbers, download the labeled replies from the dataset and run 03 to 05 on them. Running 01 and 02 again from scratch will get you close but not identical results, since Google Translate changes over time and GPU sampling isn't fully deterministic.

Dataset: https://huggingface.co/datasets/Rockeroodles/tiny-aya-indic-redteam-benchmark (gated)

## Limitations

It's small: 30 behaviors, one model and one run. Prompts and replies were translated with Google Translate and then checked by seaEagle, who reads Telugu. Obvious refusals were marked by a keyword check and we labeled the rest by hand. Over-refusal on the harmless prompts is only from the keyword check.

## References

- Chao et al. 2024. JailbreakBench: An Open Robustness Benchmark for Jailbreaking Large Language Models.
- Yong, Menghini and Bach. 2023. Low-Resource Languages Jailbreak GPT-4.
- Deng et al. 2023. Multilingual Jailbreak Challenges in Large Language Models.
- Arditi et al. 2024. Refusal in Language Models Is Mediated by a Single Direction.

## Authors

Rocks, seaEagle

## License

CC BY-NC 4.0, see LICENSE.
