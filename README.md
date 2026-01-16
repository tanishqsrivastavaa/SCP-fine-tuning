# SCP Foundation Fine-Tune (Llama 3)

A Domain Adaptation project that fine-tunes **Llama-3-8B** to generate clinical, scientific containment reports in the style of the **SCP Foundation Wiki**.

This project demonstrates a full GenAI pipeline: from data extraction and cleaning to fine-tuning with LoRA and exporting for local inference.

## 🛠️ Tech Stack
- **Model:** Llama-3-8B (Quantized 4-bit)
- **Training Framework:** [Unsloth](https://github.com/unslothai/unsloth) (Optimized for T4 GPUs)
- **Technique:** QLoRA (Quantized Low-Rank Adaptation)
- **Data Processing:** Python, BeautifulSoup4, Pandas
- **Inference Engine:** Ollama (GGUF format)

---

## 🚀 The Pipeline

### 1. Data Acquisition
We utilized the pre-processed SCP API dump to avoid scraping raw Wikidot pages.
- **Source:** [scp-data/scp-api](https://github.com/scp-data/scp-api)
- **Selection:** Filtered for `Series 1` through `Series 8` (Mainline SCP articles only).

### 2. ETL (Extract, Transform, Load)
Raw JSON data contained HTML tags and Wikidot syntax. We built a Python script to cleaning the data into a standard instruction format.

**Transformation Steps:**
1.  **Parse HTML:** Used `BeautifulSoup` to strip `<div>`, `<p>`, and image blocks.
2.  **Clean Whitespace:** Removed excessive newlines and artifacts.
3.  **Format:** Converted to **Alpaca Instruction Format** (JSONL).

**Sample Data Structure:**

{
  "instruction": "Write a clinical Special Containment Procedures report for the following object.",
  "input": "SCP-173",
  "output": "Item #: SCP-173\nObject Class: Euclid\nSpecial Containment Procedures: ..."
}

### 3. Fine-Tuning

Trained on Google Colab (T4 GPU) using Unsloth.

    Base Model: unsloth/llama-3-8b-bnb-4bit

    LoRA Config: Rank (r)=16, Alpha=16

    Training Steps: 60 steps (Proof of Concept) / 1 Epoch (Full Training)

    Loss: Converged from ~1.8 to ~0.6

### 4. Export & Quantization

The fine-tuned adapters were merged and converted to GGUF (Q4_K_M) format for efficient local inference on consumer hardware.
💻 Local Deployment (Ollama)

To run this model on your local machine (Arch Linux / Mac / Windows):

    Install Ollama:
    Bash

sudo pacman -S ollama  # Arch Linux

Create a Modelfile: Create a file named Modelfile in the same directory as your .gguf file:
Dockerfile

FROM ./scp-llama3-v1.gguf
PARAMETER temperature 0.7
SYSTEM "You are a specialized SCP Foundation database AI. Write in a clinical, detached, scientific tone. Use [REDACTED] or ████ for sensitive dates/locations."

Build the Model:
Bash

`ollama create scp-writer -f Modelfile`

Run Inference:
Bash
```
    ollama run scp-writer "Write a report for SCP-7000: A vending machine that dispenses luck."
```
🔮 Future Improvements

    [ ] Full Training: Train for 1 full epoch on the complete 9,000+ dataset to reduce repetition loops.

    [ ] Regex Cleaning: Implement stricter regex to remove "Licensing/Citation" footers from the training data.

    [ ] FastAPI Integration: Build a backend service to serve the model via REST API.