# The Small Model Infrastructure Nobody Built (So We Did)

| Field | Details |
| --- | --- |
| **Speaker** | Filip Makraduli |
| **Affiliation** | Superlinked |
| **Type** | Talk |
| **Date** | 2026-05-05 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=qdh_x-uRs9g) |
| **Tags** | `embedding-inference` `small-models` `gpu-infrastructure` `model-loading` `hot-swapping` `superlinked` |

## Summary

Filip Makraduli from Superlinked addresses a critical, often overlooked gap in the AI ecosystem: efficient inference infrastructure for small models. He recounts his personal journey from realizing his oversight in understanding real-world model speed to actively building a solution. The core problem identified is that most existing embedding infrastructure assumes a static, known model, leading to significant inefficiencies when dealing with the dynamic, multi-model requirements of modern AI applications, especially in areas like AI search and document processing.

The talk highlights that simply "chucking in more GPUs" is not the answer for small models. These models, often only a few gigabytes in size, lead to vast amounts of idle GPU memory if each model is provisioned its own GPU. This inefficiency necessitates a system capable of dynamic model loading, hot-swapping, and intelligent memory-aware eviction policies to maximize GPU utilization and reduce costs. Furthermore, the market lacked an open-source, end-to-end solution that bridges the gap from model inference to scalable production deployment, including routing, auto-scaling, and monitoring.

Superlinked's open-source Superlinked Inference Engine (SIE) was developed to fill this void. It adopts a holistic "yin and yang" approach, combining robust support for a diverse range of open-source model architectures with a sophisticated, auto-scaling infrastructure. SIE tackles the complexities of varying model runtimes and forward pass implementations while providing an integrated cluster management solution that handles hardware provisioning, load balancing, and dynamic resource allocation, making it easier for developers to deploy and manage small models at scale.

## Detailed Notes

### The Problem: Overlooked Inference for Small Models
- **Initial Oversight**: Speaker initially overlooked the critical aspect of inference in determining real-world model speed, focusing more on training and fine-tuning
- **Market Gap**: A recognized gap in infrastructure specifically designed for small model inference, particularly relevant for AI search and document processing
- **Traditional Assumptions**: Most existing embedding infrastructure assumes users know exactly which model they want ahead of time, leading to static provisioning

### Superlinked Inference Engine (SIE)
- **Open-Source Solution**: Superlinked's open-sourced inference engine (SIE) aims to address the identified gap
- **Target Use Cases**: Designed for small models used in AI search and document processing
- **Validation**: Tested and validated with major vector databases including Chroma, Quadrant, Weaviate, and LanceDB

### Why Small Model Inference Matters for Agentic Workflows
- **Context Rot**: A significant challenge where the quality of agent outputs degrades as the input context (token count) increases
  - Research paper from Chroma highlighted this effect
  - Small models are crucial for context management, preprocessing data, and filtering inputs to reduce context size
- **Tool Calling**: Small models can act as specialized tools within agentic workflows
  - **Andrej Karpathy's Approach**: Building knowledge bases using graph-based methods, leveraging Named Entity Recognition (NER) models to generate ontologies and structure file systems
  - **Chroma's Model**: Shipped their own model for preprocessing and filtering input to manage context
  - **Community Solutions**: Many efforts focus on lowering context and reducing token amounts for agent effectiveness
- **Production Use Case**: Superlinked used SIE for tool calling in an e-commerce store for taxonomy classification

### Misconceptions About Inference (What it's NOT)
- **More GPUs ≠ Solution**:
  - Small models (e.g., Stella embedding model, rerankers, Glyner NER) occupy only a few gigabytes of memory
  - "If you provision a GPU for each model, you're wasting a lot of idle space." GPUs remain idle and underutilized
  - SIE enables hot-swapping of multiple models onto a single GPU to achieve much higher utilization, lowering costs
  - Implements a Least Recently Used (LRU) eviction policy for efficient model switching
- **Only Server-Side Focus**:
  - The market lacks open-source solutions that provide a complete path from model inference to production at scale
  - Users typically have to write custom code for routing, auto-scaling, monitoring (e.g., Prometheus metrics, Grafana), queuing mechanisms, and GPU provisioning
  - SIE's goal is to fill this gap by including routing, auto-scaling, queuing, and GPU provisioning mechanisms

### The "Yin and Yang" of Inference (Superlinked's Holistic Approach)

#### Yin: Comprehensive Model Support
- **Importance of Open-Source Models**:
  - Millions of models available on Hugging Face, rapidly advancing in both size and accuracy
  - Open-source models often outperform managed services for specific, narrow tasks (validated by MTEB and other benchmarks)
  - Gemma's low-parameter models demonstrate high ELO scores, rivaling much larger models
- **Challenge of Diverse Architectures**:
  - Models like BERT, Qwen, and modern BERT have fundamentally different runtimes and implementations
  - Variations in flash attention, positional embeddings, normalization, and query-key-value (QKV) fusion
  - "There is no universal engine that can handle BERT, Qwen, and modern BERT as well because their architectures are different."
- **SIE's Solution: Re-implementing the Forward Pass**:
  - Adapts attention mechanisms, handles padding for variable length attention, and manages QKV fusion
  - **Normalization**: Handled differently in BERT, Qwen, and ColBERT
  - **QKV Fusion**: Grouped query attention in Qwen prevents QKV fusion, requiring specific handling
  - **Positional Embeddings**: BERT uses absolute lookup, while Qwen employs rotary positional embeddings
  - Support for late interaction models (e.g., ColBERT with multiple vector outputs) and cross-encoders/rerankers (which output scores instead of vectors)
  - **Variable Length Flash Attention**: Crucial for token-based batching to avoid wasting compute on empty tokens from padding

#### Yang: Robust Infrastructure
- **API Primitives**: SIE exposes three core API primitives: `encode`, `score`, and `extract`
- **Infrastructure Layer**:
  - Dynamic router and queuing mechanisms adjust based on workload
  - Utilizes different pools of GPUs and resources, including spot instances and larger GPUs
  - Implements KEDA auto-scaling with Prometheus metrics to dynamically provision hardware, prevent idle GPUs, and optimize resource usage
- **End-to-End Solution**: Provides both the cluster and comprehensive model support, eliminating the need for users to merge disparate components
- **Simplified Deployment**: Models are configured via a simple config, deployed using Terraform apply
- **Deployment Assets**: Published Helm charts and Docker images for easy deployment

### Conclusion
- **SIE Launch**: Soft launch of the Superlinked Inference Engine (SIE)
- **Open-Sourced Components**: Both the model inference capabilities and the cluster infrastructure are open-sourced
- **Background Slide Reveal**: The background image depicts a vector visualization of sinusoidal positional encoding, a foundational concept in how transformers encode position

## Key Technical Concepts

- **Embedding Inference**: The process of generating numerical vector representations (embeddings) from input data using pre-trained models, crucial for tasks like similarity search
- **Small Models**: ML models characterized by compact size (a few gigabytes in memory) compared to LLMs, often specialized for specific tasks
- **Hot-Swapping**: Dynamically loading and unloading different models onto a single GPU without service interruption
- **LRU Eviction Policy**: A cache management algorithm that removes least recently accessed models to free up memory
- **Context Rot**: Degradation of agent output quality as input context (token count) increases
- **Tool Calling**: An AI agent invoking specialized external models or functions to perform specific tasks
- **Variable Length Flash Attention**: Optimized attention handling for sequences of varying lengths, preventing wasted computation on padded tokens
- **Positional Embeddings**: Techniques to encode token position within a sequence (absolute lookup, rotary positional embeddings)
- **QKV Fusion**: An optimization combining query, key, and value linear projections into a single operation
- **KEDA Auto-scaling**: Kubernetes-based Event-Driven Autoscaling based on metrics from sources like Prometheus

## Practical Applications

- **AI Search and Retrieval**: Powering semantic search by generating embeddings for documents and queries
- **Document Processing**: Preprocessing text, extracting key information, classifying documents, managing context for downstream agents
- **Agentic Workflows**: Providing specialized small models for context management, tool calling, and reducing token usage for LLMs
- **E-commerce Taxonomy Classification**: Automating product categorization using small models as dedicated classification tools
- **Knowledge Graph Construction**: Using NER models to identify entities and relationships for structured knowledge bases

## Notable Quotes

- "If you provision a GPU for each model, you're wasting a lot of idle space."
- "There is no universal engine that can handle BERT, Qwen, and modern BERT as well because their architectures are different."
- "It's about being able to provision the hardware and also have the metrics to auto scale this."

## Timestamps

| Time | Topic |
|------|-------|
| 0:00 | Introduction and the gap in small model inference |
| 0:53 | Moving from research to building inference infrastructure |
| 2:54 | Introduction of the Superlinked inference engine |
| 4:34 | The importance of context management for agents |
| 7:03 | Misconceptions: Why more GPUs isn't the only answer |
| 9:33 | The "Yin and Yang" of inference |
| 10:43 | The challenge of supporting diverse model architectures |
| 14:33 | Deep dive into infrastructure and scalability |
| 16:10 | Conclusion and the open-source launch of SAI |

## Takeaways

- Embedding inference needs infrastructure built for dynamic multi-model serving
- Memory-aware model management is more impactful than raw GPU count
- The SAI open-source engine addresses a real gap in the ecosystem
- Small model serving has different infrastructure requirements than LLM serving

## Speaker Links

- LinkedIn: https://www.linkedin.com/in/filipmakraduli/
