Use it like this:

Download the notebook: AI Model Intake Identity Card
Open Google Colab → File → Upload notebook → upload the .ipynb.
Run the cells from top to bottom using Runtime → Run all.
In the cell named Run it, change only this line:
MODEL_INPUT = "openai/whisper-large-v3"

For example:

MODEL_INPUT = "Qwen/Qwen2.5-VL-3B-Instruct"

or:

MODEL_INPUT = "microsoft/trocr-base-printed"
Run that cell. The notebook will automatically fetch Hugging Face metadata and generate:
model name / organization
LLM, VLM, OCR, Audio, Embedding, etc.
input and output type
intended purpose
architecture / base model
license / datasets
parameter count where available
suspicious model file formats
custom-code indicators
similar models doing the same task
recommended security testing route
At the bottom you'll see paths such as:
/content/model_identity_cards/...

The main output is a JSON Model Identity Card. You can download it from Colab's Files panel.

For initial testing, try these:

openai/whisper-large-v3                   # Audio / ASR
Qwen/Qwen2.5-VL-3B-Instruct              # VLM
microsoft/trocr-base-printed              # OCR
sentence-transformers/all-MiniLM-L6-v2    # Embedding
google-bert/bert-base-uncased             # NLP

For public models, no Hugging Face token is required. For private/gated models, add your HF_TOKEN in Colab Secrets.

Most importantly, this notebook does not download or execute the actual model weights. It performs the identity/classification step first, which is exactly what you want before moving the model into quarantine.
