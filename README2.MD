What this version does
Accepts a model name or Hugging Face ID. It accepts openai/whisper-large-v3, a Hugging Face URL, and—only for experimentation—a short name. For production I added REQUIRE_EXACT_MODEL_ID = True, because letting a gateway guess which repository someone meant is a security mistake.
Creates the Identity Card without loading the model. It calls Hugging Face metadata APIs and fetches only small metadata such as README.md and config.json. It deliberately does not call AutoModel.from_pretrained, torch.load, pickle.load, or trust_remote_code=True. Hugging Face itself warns that pickle-based model files can execute arbitrary code when deserialized.
Classifies the model automatically. The main signal is Hugging Face's pipeline_tag, which is specifically intended to describe the model's task and is used for Hub discovery. I then add architecture/name/tag heuristics for cases where metadata is incomplete. Current families include LLM, VLM, OCR/Document AI, Audio, Computer Vision, Generative Vision, Embedding, Reranker, NLP, and Manual Review.
Determines input/output and purpose. For example, ASR becomes audio → text, VLM becomes image + text → text, OCR becomes document/image → text, embedding becomes text → vector, and so on.
Builds supply-chain information. It captures repository owner, declared developer when available, commit SHA, base model, architecture, library, license, training datasets, languages, parameter count when discoverable, context length, timestamps, gated/private state, tags, downloads, and likes. Hugging Face model cards natively support standardized metadata such as license, datasets, base model and pipeline task.
Examines the repository attack surface before execution. It identifies .pkl, .pickle, .joblib, .pt, .pth, .bin, .ckpt, H5/Keras files, Safetensors/GGUF/ONNX files, Python code, dependency manifests, modeling_*.py, configuration_*.py, and auto_map. The current Hugging Face API can also request repository security-status information and file metadata without loading the model.
Finds alternative models automatically. One thing I would change from your assumption: I could not verify a first-class “alternative models doing the same job” feature in the current OWASP AIBOM generator documentation. So I implemented it ourselves. Hugging Face's API explicitly supports searching models by pipeline_tag and sorting them, so the notebook finds other models performing the same standardized task and returns the top comparable candidates. These are discovery candidates, not automatically safer models.
Generates a security test route. This is the part that turns the card from an inventory record into something operational. Every model gets common provenance/artifact checks, then family-specific tests. LLMs route toward jailbreak/prompt-injection/data-leakage testing; VLMs toward multimodal injection; OCR toward malicious document/layout tests; audio toward spoken/embedded instruction and parser testing; vision toward adversarial input testing; embeddings toward semantic manipulation testing. I included ModelScan where model serialization scanning applies, and garak/PyRIT where generative red-teaming is appropriate. ModelScan is specifically designed to detect unsafe serialized model content; garak targets LLM vulnerabilities; PyRIT supports red-team workflows and text/audio/image/video converters.
Adds an OWASP AIBOM bridge. The output includes mappings for fields such as primaryPurpose, suppliedBy, type, typeOfModel, datasets, license, component name/version and download location. I deliberately label this a mapping bridge, not a standards-valid AIBOM. When you require a formal CycloneDX AIBOM, use the OWASP generator/CycloneDX library rather than inventing our own supposedly compliant JSON. OWASP's current generator uses the official CycloneDX Python library and supports CycloneDX 1.6 output.
Includes production integration hooks. There is already a handle_model_gateway_request() example and a separate policy-gate example, so your eventual architecture can become:
Model Request
     ↓
Model Gateway
     ↓
Identity Card Service  ← no model execution
     ↓
Model Identity JSON
     ↓
Policy Gate
     ↓
Quarantine Download
     ↓
Static Model / Malware / Dependency Scan
     ↓
Family-Specific Security Tests
     ↓
Approve / Reject / Manual Review
