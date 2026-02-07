# Multimodal Visual Security Engine (EasyOCR + ONNX DeBERTa + CLIP + BLIP)

## System Architecture

```mermaid
graph TD
    Input[Input: Image/Video Frame] --> Split{Parallel Process}

    %% Engine D Logic
    Split --> EngineD[Engine D: Prompt Injection]
    EngineD --> OCR[EasyOCR: Extract Text]
    OCR --> Norm[Normalization Layer]
    Norm --> InjectModel[DeBERTa Prompt Injection (ONNX)]
    InjectModel --> ThreatCheck{Threat Dictionary (aux)}
    ThreatCheck --> RiskScore[Risk Score + Reason]

    %% Engine E Logic
    Split --> EngineE[Engine E: Cross-Modal]
    EngineE --> BLIP[BLIP: Image Caption]
    InputAudio[Input: Audio Transcript] --> CLIP_Text[CLIP Text Encoder]
    EngineE --> CLIP_Img[CLIP Image Encoder]
    CLIP_Text --> Cosine[Cosine Similarity Calc]
    CLIP_Img --> Cosine
    Cosine --> Threshold{Is Score < 0.18?}
    Threshold -- Yes --> Mismatch[Status: MISMATCH - Deepfake]
    Threshold -- No --> Match[Status: MATCH - Genuine]
```

**Engine D (Visual Prompt Injection)**  
OCR-based text extraction + ML classification. EasyOCR extracts visible or hidden text (with CLAHE + Otsu binarization for low-contrast regions), a normalization layer de-obfuscates tokens, and a DeBERTa prompt‑injection classifier (ONNX runtime) scores risk. A small threat dictionary is used as auxiliary evidence in the reason string, not as the primary detector.

**Engine E (Cross-Modal Consistency)**  
Semantic-based (not OCR). CLIP (ViT-B/32) embeds both the video frame and the audio transcript into a shared vector space to verify that the visual context matches the spoken context. BLIP generates an image caption and we compare it with OCR text to detect prompt/scene misalignment.

## Quick Start

```bash
# Install dependencies
pip install -r requirements.txt

# Run the Visual Engine Test
python -m src.engines.visual_engine
```
