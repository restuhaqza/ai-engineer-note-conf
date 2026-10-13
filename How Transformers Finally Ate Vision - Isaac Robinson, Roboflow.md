# How Transformers Finally Ate Vision

| Field | Details |
| --- | --- |
| **Speaker** | Isaac Robinson |
| **Affiliation** | Roboflow |
| **Type** | Talk |
| **Date** | 2026-05-08 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=VhfAVA3BG2I) |
| **Tags** | `vision` `transformers` `vit` `cnns` `computer-vision` `vlms` `world-models` |

## Summary

Isaac Robinson, Research Lead at Roboflow, delves into the paradigm shift in computer vision, explaining how Transformers, initially perceived as an unnatural fit for image data, ultimately surpassed the long-dominant Convolutional Neural Networks (CNNs). The talk traces the evolution from early Vision Transformers (ViT) through architectural innovations like Swin and ConvNeXt, ultimately demonstrating that massive-scale pretraining, particularly with ViT-specific methods like Masked Autoencoders (MAE), proved more impactful than architectural inductive biases.

Robinson argues that the success of Transformers in vision was significantly accelerated by leveraging the vast infrastructure, tooling, and optimizations developed for large language models (LLMs). This shared ecosystem, combined with the ability of simpler, scalable architectures to learn complex visual patterns through extensive self-supervised pretraining, allowed Transformers to overcome their initial computational hurdles and lack of inherent visual inductive biases. The presentation culminates in a discussion of the practical implications for foundation models like SAM and Roboflow's efforts to address the critical challenge of deployment flexibility in real-world, resource-constrained environments.

## Key Insights

### The CNN-to-Transformer Arc
- CNNs were dominant because their inductive biases (locality, translation invariance) matched image structure well
- Transformers looked like a less natural fit for images but won through **scale, pretraining, and shared infrastructure with LLMs**

### Model Evolution Timeline
- **ViT** (Vision Transformer) -- proved transformers could work for images
- **Swin** -- added hierarchical processing
- **ConvNeXt** -- showed modernized CNNs could match transformers
- **Hiera** -- simplified architecture without hand-designed components
- **SAM** (Segment Anything) -- foundation model for segmentation
- **RF-DETR** -- real-time detection transformer

### Why Transformers Won
- **Pretraining** at scale was more impactful than architectural inductive biases
- **Shared infrastructure** with the LLM world accelerated development
- Deployment flexibility now matters as much as raw benchmark wins

### What's Next
- **VLMs** (Vision-Language Models)
- **World models** for physical AI
- The convergence continues toward unified architectures

## Detailed Notes

### The Initial Competition: CNNs vs. Transformers

*   **Convolutional Neural Networks (CNNs):**
    *   **Inductive Bias:** Excellent inductive biases (locality, translation invariance) motivated by the human visual system. A feature (e.g., a person) is recognized regardless of its position in an image.
    *   **Architecture:** Employ filters convolved across images, building hierarchical structures (e.g., ResNets).
    *   **Compute Scaling:** Typically N-squared with image resolution.
*   **Transformers:**
    *   **Operation:** A set-to-set operation with no inherent inductive bias. Inductive biases are injected (e.g., causal mask for autoregressive language models).
    *   **Vision Transformer (ViT):**
        *   **Method:** Image is split into fixed-size patches (e.g., 16x16), a learned positional encoding is added, and these tokens are fed into a standard transformer.
        *   **Compute Scaling:** N-squared on patches, leading to N-to-the-fourth on pixel resolution.
        *   **Inductive Bias:** No inherent locality or translation invariance; a feature in the upper left can have a totally different activation pattern if in the bottom right.
*   **The Core Question:** How did the high-inductive-bias, N-squared CNN lose to the no-inductive-bias, N-to-the-fourth ViT?
*   **Answer:** Massive ViT-specific pretraining and borrowing speedups/infrastructure from the LLM boom.

### The Evolution of Vision Transformer Architectures

1.  **ViT (Vision Transformer):**
    *   **Contribution:** Proved that transformers could work for images despite their initial disadvantages.
2.  **Swin Transformer:**
    *   **Motivation:** Address ViT's N-to-the-fourth compute scaling and lack of locality.
    *   **Method:** Instead of global attention, uses "windowed attention" within local regions. To allow interaction between windows, it employs "shifted windows" in subsequent layers, mimicking the overlap of convolutional filters.
    *   **Result:** Achieves N-squared compute (if window size is independent of resolution) and reintroduces a locality inductive bias.
3.  **ConvNeXt:**
    *   **Motivation:** Can modern CNNs, borrowing transformer design principles, compete?
    *   **Method:** Takes learnings from ViTs (e.g., patchify operation, mixer-feedforward block structure, hierarchical design, LayerNorm) and applies them to a convolutional network. Uses a 4x4 patchify. Convolution acts as the "mixer" for spatial information.
    *   **Result:** Beats ViT and Swin on standard ImageNet benchmarks.
4.  **Hiera (Meta):**
    *   **Motivation:** Identify the truly important components. Can pretraining learn biases instead of hand-designing them?
    *   **Method:** Starts with a good inductively biased transformer, systematically strips out biases, and uses massive pretraining to recover performance.
    *   **Key Pretraining Technique:** **Masked Autoencoder (MAE)**.
        *   **Process:** Take an image, split into patches, randomly drop a large percentage of patches, and train the model to reconstruct the missing patches based on context.
        *   **Analogy:** Very similar to BERT's masked language modeling.
        *   **Specificity:** MAE is ViT-specific; it cannot be applied to CNNs due to their inherent translation invariance.
    *   **Result:** MAE at scale enables ViTs to learn inductive biases that would otherwise be missing from their structure.
    *   **Further Pretraining:** DINOv2, DINOv3 are advanced ViT-specific self-supervised pretraining techniques that produce incredibly rich, semantically meaningful feature maps (e.g., PCA decomposition of DINOv3 features accurately delineates cat paws or satellite imagery components).
    *   **Performance:** Self-supervised learning objectives (via linear probe on frozen features) are catching up to fully supervised learning performance.

### The Role of LLM Infrastructure and Speedups

*   **The Speed Problem:** Despite architectural improvements, ViTs still faced the N-to-the-fourth compute scaling issue.
*   **Solution:** Innovations from the LLM world, particularly **Flash Attention**, significantly optimize the attention mechanism.
*   **Impact:** When Flash Attention is applied, the speed advantage of architectures like Hiera (which explicitly noted speedups without Flash Attention) diminishes, bringing the "silly N-to-the-fourth thing" (ViT) back into contention due to its benefits from ViT-specific pretraining.

### Practical Applications and Deployment Flexibility

*   **SAM (Segment Anything Model):**
    *   **Evolution:** SAM -> MobileSAM -> SAM2 -> SAM3.
    *   **Backbones:**
        *   SAM: ViT trained with MAE.
        *   MobileSAM: Specialized convolutional-transformer hybrid (TinyViT).
        *   SAM2: Hiera with MAE pretraining.
        *   SAM3: Returns to a "massively pre-trained backbone," abandoning architectural ablation for raw pretraining power.
    *   **Observation:** The trend for foundation models converges back to massively pre-trained, ViT-like backbones.
*   **The Deployment Flexibility Challenge:**
    *   **Issue:** Large foundation models (e.g., SAM3: 800 million parameters, 300ms inference on a T4 GPU) are expensive and slow, making them unsuitable for low-power, edge, or resource-constrained devices where vision historically operates. They are "one-size-fits-all" but not flexible.
*   **Roboflow's Solution: RF-DETR:**
    *   **Goal:** Transform fixed foundation models into flexible, deployable solutions.
    *   **RF100VL Dataset:** Introduced to measure how well foundation models transfer to diverse downstream object detection tasks.
    *   **Performance:** Achieves a 40x speedup for the same accuracy compared to fine-tuning SAM3, and a meaningful improvement for a 15x speedup.
    *   **Method:** Uses **Neural Architecture Search (NAS)** to modify existing foundation models, generating entire families of high-performance models.
    *   **Flexibility:** Introduces "flexible knobs" that are drop-in compatible with existing foundation model infrastructure, allowing customization based on target data and hardware.
    *   **Impact:** This combination of massive ViT-specific pretraining, LLM-driven speedups, and pretraining-compatible NAS is the "final nail in the coffin" for classical convolutional methods, enabling deployment in hardware-constrained environments. RF-DETR outperformed state-of-the-art real-time instance segmentation models.

### What's Next (Q&A)

*   **Video Processing:** SAM3 incorporates video processing for object tracking.
*   **Unified Architectures:** Continued work on models like JEPA and V-JEPA for video and multi-modal pretraining, though their downstream transfer capabilities are still evolving.

## Key Technical Concepts

*   **Inductive Bias:** Assumptions built into a model's architecture that guide its learning process. For CNNs, these include **locality** (nearby pixels are related) and **translation invariance** (a feature is recognized regardless of its position). Transformers, by default, lack these.
*   **Convolutional Neural Networks (CNNs):** A class of deep neural networks primarily used for analyzing visual imagery. They use convolutional layers to automatically and adaptively learn spatial hierarchies of features from input data.
*   **Transformers:** An architecture introduced for sequence-to-sequence tasks (like language translation) that relies on self-attention mechanisms to weigh the importance of different parts of the input sequence.
*   **Vision Transformer (ViT):** The first successful application of the Transformer architecture directly to image classification. It treats an image as a sequence of flattened patches, each patch becoming a "token."
*   **Swin Transformer:** A hierarchical Vision Transformer that addresses ViT's quadratic complexity with respect to image size by computing self-attention within local "windows" and allowing cross-window connection through shifted windows.
*   **ConvNeXt:** A modern convolutional neural network architecture that incorporates design principles from Vision Transformers (e.g., larger kernel sizes, inverted bottleneck, LayerNorm) to achieve competitive performance.
*   **Masked Autoencoder (MAE):** A self-supervised pretraining method for Vision Transformers where a large portion of image patches are masked, and the model is trained to reconstruct the original pixel values of the masked patches. This forces the model to learn rich, contextual representations.
*   **DINOv2/DINOv3:** Advanced self-supervised learning methods for Vision Transformers that learn robust visual features without labels, often by distilling knowledge from a "teacher" network to a "student" network. They produce highly semantic feature maps.
*   **Flash Attention:** An optimized attention algorithm for Transformers that significantly reduces memory usage and speeds up computation by reordering operations and avoiding materializing large intermediate matrices.
*   **Foundation Models:** Large-scale, pre-trained models (often multi-modal) that can be adapted to a wide range of downstream tasks with minimal fine-tuning. Examples include SAM for vision and large language models like GPT-3.
*   **Neural Architecture Search (NAS):** An automated technique for designing neural network architectures. It explores a search space of possible architectures to find the one that performs best on a given task, often optimizing for both accuracy and efficiency.

## Practical Applications

*   **Prioritize Pretraining and Scale:** For new vision tasks, focus on leveraging large-scale pretraining (especially self-supervised methods like MAE or DINO for ViTs) rather than solely relying on hand-crafted architectural inductive biases. The ability to scale and learn from massive datasets is paramount.
*   **Leverage LLM Infrastructure:** When working with Transformers in vision, actively seek out and utilize optimizations and tooling developed for LLMs (e.g., Flash Attention, efficient inference libraries). The shared ecosystem provides significant compounding advantages.
*   **Consider Deployment Flexibility Early:** For real-world applications, especially on edge devices or with strict latency requirements, raw benchmark accuracy is not enough. Plan for deployment flexibility from the outset, considering model size, inference speed, and hardware constraints.
*   **Utilize Foundation Models Strategically:** Foundation models like SAM offer powerful general-purpose capabilities. However, be aware of their computational cost and parameter count. For specific, resource-constrained deployments, they may need significant adaptation.
*   **Explore Neural Architecture Search (NAS):** To bridge the gap between powerful foundation models and practical deployment, consider using NAS. It can help adapt large models to specific hardware and data requirements, generating efficient model families without sacrificing too much accuracy.

## Notable Quotes

*   "The thing that is in the upper left can have a totally different activation pattern if it's in the bottom right. And so, the question naturally arises, which is better? The high inductive bias n-squared convolutional network or the no inductive bias n to the fourth power VIT? So, as everyone would expect, it's the VIT."
*   "This is I think a really really great example of the balance between pretraining and inherent inductive bias, which ends up being how transformers ultimately win out."
*   "This combination of this huge advancement in foundation model pre-training and these strong deployment methods combined with an actual ability to deploy these in hardware constrained environments ends up being the final nail in the coffin for these classical convolutional-based methods."

## Takeaways

- Inductive biases matter less than scale and pretraining
- Infrastructure shared with NLP gave vision transformers a compounding advantage
- The evolution shows a pattern: simpler architectures that scale outperform hand-crafted ones
- Deployment flexibility is now a first-class concern alongside accuracy

## Speaker Links

- LinkedIn: https://www.linkedin.com/in/robinsonish/
