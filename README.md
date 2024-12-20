# Eden: Small Multi-Modal 

## 🎯 Objective: 


The project aimed to finetune a multi-modal language model that accepts text, image, and audio inputs and generates text output based on a query.

---

## 📑 Dataset:

- Instruct 150k: A carefully curated dataset designed to enhance the model’s instruction-following capabilities.
- COCO Images: Original images from the COCO dataset were paired with text data to create a rich, multi-modal learning environment.

---
## [Demo](https://huggingface.co/spaces/HimankJ/Eden-Multimodal)

### Image Input

<img width="1267" alt="image" src="https://github.com/user-attachments/assets/2a6971fa-ab6f-4e6c-9162-d3ad925d8fbb">

<img width="1269" alt="Screenshot 2024-10-22 at 11 42 25 PM" src="https://github.com/user-attachments/assets/0a01e7ef-a7c0-48a9-96e8-7e68928ee673">

<img width="1266" alt="Screenshot 2024-10-22 at 11 38 25 PM" src="https://github.com/user-attachments/assets/fbb56cb3-2b10-48d3-9a2d-e3ab140c4407">

### Audio Input

<img width="1302" alt="Screenshot 2024-10-23 at 12 00 40 AM" src="https://github.com/user-attachments/assets/0efab7df-f157-4cd0-96cb-f0910bedb8c1">

<img width="1264" alt="Screenshot 2024-10-23 at 12 02 53 AM" src="https://github.com/user-attachments/assets/bf02d8e3-19ba-4d47-be47-19923a6b3ecc">

### Text Input

<img width="1267" alt="Screenshot 2024-10-23 at 12 15 39 AM" src="https://github.com/user-attachments/assets/12840f79-fee9-4687-9960-9d79ec2af343">

---

## :basecamp: Model Architecture:

- Llava Model Approach: The project adopted a Llava-like framework, leveraging the CLIP model for image embeddings. These embeddings were fine-tuned for textual alignment.

- CLIP Pretrained Model:

  - Extracts image embeddings which are then adjusted to align with text representations.

- Projection Layer:

  - A transformation layer that adjusts CLIP image embeddings into a form that is compatible with the downstream language model, ensuring seamless integration with Phi3.5.

- Phi3.5_mini_instruct Model:

  - Acts as the core language model (SLM).
  - Fine-tuned using QLora for efficient instruction-tuning and to comprehend image embeddings effectively.

- Whisper for Audio:

  - Whisper model converts audio inputs into text, which is combined with other textual data and questions.
  - Both the audio-converted text and original textual inputs are passed to the Phi3.5 layer for comprehensive processing.

--- 

## 📍 Deployment:

- The final model was deployed on Huggingface, allowing users to submit multi-modal inputs (image, audio, and/or text) along with a query.
- The model generates a coherent text-based output, informed by all available input modalities and the given question.

---

## ⭐ Significance: 
This model demonstrates a flexible and scalable approach to multi-modal AI by integrating advanced models like CLIP, Phi3.5, and Whisper, enabling real-world applications such as conversational AI, visual Q&A, and multimedia interpretation.

---

## Core Components

### 1. Models and Processors

- CLIP: Used for image feature extraction
  - Model: openai/clip-vit-base-patch32
  - Processor: Handles image preprocessing

- Phi-3.5: Base language model
  - Model: microsoft/Phi-3.5-mini-instruct
  - Configured with 4-bit quantization for memory efficiency
  - Uses bfloat16 for computation

### 2. Data Collation

- Batching of image features
- Tokenization of text segments
- Creation of attention masks
- Label generation for causal language modeling
- Special handling of answer sections for loss calculation

### 3. Projection Block

- Projects CLIP image features into Phi's embedding space:
```
Input (512) → LayerNorm → Linear → GELU → Linear → Output (3072)
```

- MultimodalPhiModel
  - Core multimodal architecture that:
  - Inherits from PreTrainedModel
  - Combines projected image features with text embeddings
  - Handles model saving/loading
  - Processes both start and end text segments

### 4. Training Implementation

  - Model Configuration
    - Uses LoRA for efficient fine-tuning
    - Parameters:
      - lora_alpha: 16
      - lora_dropout: 0.1
      - r: 64
       
    - Targets specific modules:
      - o_proj
      - qkv_proj
      - gate_up_proj

  - Training Configuration
    - Batch size: 2 (per device)
    - Maximum steps: 6000
    - Save frequency: Every 40% of training
    - Logging frequency: Every 10% of training
    - Evaluation frequency: Every 10% of training
    - Uses bfloat16 precision
    - Implements gradient checkpointing

### 5. Memory and Performance Optimizations

  - 4-bit quantization for the base model
  - Gradient checkpointing enabled
  - LoRA for efficient fine-tuning
  - bfloat16 precision training
  - Selective parameter updates

---

## Data Flow

1. Image → CLIP → Image Features (512 dim)
2. Image Features → ProjectionBlock → Projected Features (3072 dim)
3. Text → Tokenizer → Token Embeddings
4. Concatenation: [Start Text Embeddings + Projected Image Features + End Text Embeddings]
5. Combined Input → Phi-3.5 → Generated Response

---

## Logs

Training Loss

<img width="1434" alt="image" src="https://github.com/user-attachments/assets/fe770a3a-7766-4511-a85a-7e6487d467e3">

---

## Architectural Limitations

1. **Projection Layer Constraints**
  - Current simple two-layer projection may be insufficient for complex cross-modal alignment

2. **Memory Management Issues**
  - Large batch sizes not possible due to:
    - Image embeddings taking significant memory
    - Full attention computation being memory intensive
    - Combined text and image tokens creating long sequences

  - Gradient checkpointing helps but impacts training speed

3. **Training Inefficiencies**
  - Small batch size limits effective gradient updates
  - Fixed learning rate may not be optimal for different training phases
  - Limited data augmentation for multimodal inputs

---

## Future Scope (What can be done better)

1.) **Diversifying Data Sources for Better Generalization** - 
The current implementation could benefit significantly from incorporating diverse data sources beyond the existing dataset. Introducing varied image-text pairs from different domains, contexts, and styles would help the model develop more robust and generalizable representations.

2.) **Granular Image Processing Approach** - 
Rather than processing images as single units, breaking them down into smaller, meaningful segments could enhance the model's understanding of visual details. This approach would allow the model to focus on specific image regions and establish stronger connections between image components and textual descriptions. The implementation could involve using sliding windows, attention-based region selection, or semantic segmentation to divide images.

3.) **Flash Attention Implementation** -
The integration of Flash Attention would mark a significant improvement in computational efficiency and memory usage. This would enable processing of longer sequences and larger batch sizes, potentially leading to better training dynamics.

4.) **Enhanced Architecture for Cross-Modal Understanding** -
The current projection layer architecture could be enhanced to create more sophisticated interactions between image and text modalities. Implementation of cross-attention mechanisms, hierarchical projection layers, and modality-specific preprocessing steps would allow for more nuanced feature extraction and combination.

5.) **Extended Training Paradigm** -
While current GPU limitations constrain training duration and batch size, the model's performance could potentially improve significantly with longer training periods and larger batch sizes. (😢)

6.) **Alternative Vision Model Integration** -
The exploration of different vision models, particularly the integration of Qwen or similar advanced vision models, could provide alternative approaches to visual feature extraction.

---
