# How Google DeepMind is researching the next Frontier of AI for Gemini

| Field | Details |
| --- | --- |
| **Speaker** | Raia Hadsell |
| **Affiliation** | Google DeepMind |
| **Type** | Talk |
| **Date** | 2026-04-18 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=zZsTVBXcbow) |
| **Tags** | `rag-context-and-search` `architecture` `robotics-and-world-models` |

## Summary

Raia Hadsell, VP of Research at Google DeepMind, gives a tour of frontier AI work
that goes beyond language models. She frames her group's method as finding **root
problems**: deep, unsolved issues whose resolution unlocks many downstream
applications. She then picks three examples, each exercising a different
capability.

The first is representation learning. Drawing on the neuroscience idea of a
"Jennifer Aniston cell" — a small set of neurons that respond to one person across
sight, sound, and text — she explains embedding models trained with contrastive
losses. Gemini Embedding 2 is presented as a unified, "omnimodal" model that puts
text, images, video, audio, and PDFs into one semantic space. Matryoshka
Representation Learning (MRL) then lets a single network produce vectors of
adjustable size.

The second example is weather prediction. GraphCast uses a spherical graph neural
network to forecast the whole atmosphere; GenCast makes the forecast probabilistic,
capturing the extreme tails that matter for risk; and FGN trains cyclone
properties directly instead of bolting on a detector afterward. The third is world
models: Genie 1 could make short 2D playable worlds, Genie 2 moved to interactive
3D, and Genie 3 produces high-quality worlds that respond to a body in the scene,
remember earlier places, and can be re-prompted while you are inside them.

## Key Takeaways

- **Look for root problems.** Solving one deep, foundational issue can enable many
  downstream applications.
- **Embeddings are the companion to generation.** Sometimes you generate something
  new; sometimes you retrieve what already exists.
- **Contrastive losses teach relationships.** They pull related inputs together and
  push unrelated ones apart, robustly across modalities.
- **One vector, many inputs.** Gemini Embedding 2 encodes text, images, video,
  audio, and PDFs into a single shared semantic space.
- **MRL makes vector size adjustable.** A short prefix (say 256 dimensions) gives a
  cheap first pass; the full vector reranks the shortlist.
- **GraphCast forecasts the atmosphere.** A spherical graph neural network predicts
  about 100 variables autoregressively for up to 15 days.
- **GenCast models uncertainty.** A probabilistic forecast describes the tails —
  the rare but severe outcomes that matter for risk.
- **Genie 3 worlds respond, remember, and change.** Generated environments track
  your presence, keep earlier places consistent, and can be re-prompted live.

## Detailed Notes

### A research path from philosophy to neural networks · 00:44
- Hadsell has been at DeepMind almost 13 years and is a UK AI ambassador, helping
  bridge government, academia, and industry.
- In the 1990s she studied philosophy of religion; in the 2000s she moved into
  computer science and did a PhD with Yann LeCun on convolutional networks and
  neural networks for robots.
- In the 2010s she joined a DeepMind of roughly 30–40 people, working on Atari, Go,
  StarCraft, and robotics.
- Today she helps lead about 1,200 scientists and engineers across 10 labs.
- She frames "frontier AI" as three questions: which architectures come next for
  Gemini, which problems only AI can address, and how human and robotic
  intelligence change alongside artificial intelligence.

### Find the root problem · 03:11
- Her group looks for **root nodes**: don't spend time on the leaves; find the
  deepest unsolved problem and solve it to unlock downstream impact.
- DeepMind's mission, in her words, is to build AI responsibly for the benefit of
  humanity, which includes choosing problems worth solving.
- The research agenda covers far more than language models; she deliberately picks
  three non-language examples.
- She stresses partnership: external partners help both find the root problems and
  bring the results into practical use.

### Recognize the concept, not just its presentation · 05:18
- A "Jennifer Aniston cell" is not necessarily one neuron, but a small set of
  neurons that respond to a specific person, thing, or place.
- Those cells are robust across modalities: a name, a photo, a video, or a voice
  all activate the same representation, enabling fast recognition, retrieval, and
  comparison.
- An **embedding model** tries to build the same property artificially: encode
  inputs so their relationships stay useful when the presentation changes.
- The useful output is not a new description but a representation that lets a
  system compare inputs and retrieve relevant material.
- **Contrastive losses** train those relationships, making related examples
  compatible and separating unrelated ones. Hadsell connects this to her PhD work
  on Siamese networks, an early approach to learning comparisons.
- Embeddings complement generative AI: sometimes the next step is to generate,
  sometimes to retrieve.

### One semantic space, adjustable vector sizes · 07:33
- **Gemini Embedding 2** is derived from Gemini and described as fully omnimodal,
  with strong world knowledge and retrieval quality.
- A unified multimodal space avoids stitching separate per-modality representations
  together and losing information at the joins.
- Hadsell describes one vector representing up to 8K text tokens, 128 seconds of
  video, 80 seconds of audio, or a full PDF. The published launch limits differ
  slightly (8,192 text tokens, 120 seconds of video, PDFs up to six pages), so
  those should govern input preparation.
- Uses include retrieval, querying, and agentic logic.
- **Matryoshka Representation Learning (MRL)** makes the representation's size
  adjustable: one network learns nested representations, so a shorter prefix can
  support an inexpensive retrieval pass.
- Her example starts at 256 dimensions and then expands; importantly, you are using
  more coordinates from the learned representation, not recovering coordinates that
  were discarded when only a short vector was stored.
- Practical flow: keep the full vectors available, use their first 256 coordinates
  to shortlist candidates, then rerank the shortlist with the full representation.
- The richer pass cannot rescue a candidate the short first pass excluded, so
  shortlist size remains a retrieval-quality choice.
- She calls the quality state of the art, without giving a numeric embedding
  benchmark in this talk.

### Learn to predict the atmosphere · 10:00
- The weather work started with a UK Met Office scientist asking whether AI could
  predict rainfall better than physics-based models.
- Atmospheric simulation is hard, but 40 years of global weather data made
  forecasting tractable for neural networks.
- **GraphCast** predicts the state of the atmosphere up to 15 days out (the
  original release documents 10 days), everywhere on Earth.
- Its architecture is a spherical graph neural network: nodes wrap the globe from
  the surface into the lower stratosphere.
- It takes the atmosphere's current state and predicts the next state
  **autoregressively** — each prediction becomes the next input.
- It handles about 100 variables, such as wind speed, temperature, and humidity.
  A demo shows Hurricane Lee's nine-day predicted track making landfall on Nova
  Scotia.
- She reports landfall guidance nine days ahead, versus six days for the best
  physics-based models; three extra days matters for a major hurricane.
- **GenCast** keeps the mesh-based approach but is **probabilistic**, which matters
  because weather is chaotic and decisions depend on the tails of the distribution.
- Reported figures: GenCast was more accurate on 97% of about 1,300 benchmark
  forecasts, and produced a 15-day forecast in eight minutes on one chip instead of
  hours on a large supercomputer. (The 97% counts variable-and-lead-time targets,
  and the eight-minute run is for one ensemble member.)

### Make the cyclone a direct prediction target · 13:37
- **FGN** (Functional Generative Network) predicts cyclones directly, rather than
  predicting the weather and then attaching a cyclone detector as post-processing.
- Cyclone recognition, categorization, trajectory, wind speed, and eye formation
  are trained into the network.
- The design question is where the application target enters the system: learned
  inside the model, or interpreted from its output afterward.
- She says the US National Hurricane Center has already used it and is enthusiastic;
  the talk does not specify whether that use is experimental or routine.
- Worldwide adoption is hoped for in the coming years.

### Generate the environment, not just train the agent · 14:33
- World models grow out of DeepMind's work on Atari, Go, StarCraft, and
  MuJoCo-style environments for robotics, used to study agency and the environment.
- Over time the focus widened from training the agent to creating an effectively
  unbounded supply of environments.
- **Genie 1** turned a requested world into a short 2D platformer that responded to
  left and right controls; worlds lasted only seconds, but their variety justified
  scaling the data and method.
- **Genie 2** extended the idea to interactive 3D, but interaction was too slow to
  be real time and the visuals were not realistic or high definition.
- The progression separates requirements that the word "generation" can hide:
  diversity, responsiveness, dimensionality, and visual quality each had to
  improve.
- **Veo 3** is presented as a turning point before the newer demonstrations; she
  does not give a Veo-to-Genie integration recipe, shifting emphasis to what the
  worlds now let a person do.

### Worlds you can enter, change, and remember · 16:38
- Prompt: a muddy lane in Kent. Looking down reveals a body, and walking forward
  makes the water move — the model learned engagement, not just appearance.
- Another example switches to first-person skiing with gloved hands and ski tips,
  showing the breadth of interaction the system must support.
- A video-to-world demo begins with an artist's few seconds of footage and
  continues it into an explorable world; the on-screen label changes from Original
  to Genie.
- Users can fly through, bounce off a structure, move away, and return to it, so
  places must stay available as the viewpoint changes.
- The "origami lizard in an origami world" prompt tests memory: running one
  direction for a minute and returning finds the environment unchanged.
- Genie 3's announcement describes consistency over a few minutes, so this is
  bounded memory rather than indefinite persistence.
- The last demo prompts the world while inside it, changing the environment
  repeatedly during play; the world is no longer fixed by the request that started
  it.
- She imagines a new gaming format where one person adversarially prompts another
  person's world, and education where you enter a world in order to learn about it.
- She closes by pointing to her colleague Omar's talk the next morning on Gemma 4,
  returning explicitly to language models.

## Resources

- Gemini Embedding 2 launch announcement
- Matryoshka Representation Learning (paper)
- GraphCast: global weather forecasting
- GenCast: probabilistic weather forecasting
- AI for tropical cyclone prediction (Weather Lab and cyclone evaluation)
- Genie 3: a new frontier for world models
