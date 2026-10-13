# MCP UI: Extending the Frontier

| Field | Details |
| --- | --- |
| **Speaker** | Liad Yosef and Ido Salomon |
| **Affiliation** | MCP Apps |
| **Type** | Talk |
| **Date** | 2026-05-06 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=o-zkvb0iFDQ) |
| **Tags** | `mcp` `ui` `chatgpt` `claude` `vscode` `interactive-ui` `agent-distribution` |

## Summary

Chat interfaces are no longer limited to walls of text, which were a significant blocker for companies wanting to integrate with AI agents due to loss of brand identity and interactivity. Liad Yosef and Ido Salomon, co-creators and maintainers of MCP Apps, explain how this standard allows tools to send their own rich, interactive UI directly into hosts like ChatGPT, Claude, VS Code, Cursor, and Microsoft Copilot. This moves beyond simple text responses, enabling a dynamic and branded user experience within agentic environments.

The concept originated with MCPUI in May last year, focusing on a general way to pass UI over MCP and facilitate communication between the UI and the host. This initiative quickly evolved into MCP Apps, becoming the first official extension to the MCP standard through partnerships with Anthropic and OpenAI. Early adopters included Shopify and Hugging Face, and now major platforms recommend and support MCP Apps as the standard for building interactive experiences within their ecosystems.

The session delves into the core architecture, demonstrating how a tool call can return an interactive HTML resource instead of plain text, which the host then renders securely. This bidirectional flow allows user interactions within the UI to send messages back to the host, enabling further tool calls or follow-up actions, all while maintaining context. The speakers envision a "new web" where personal assistants orchestrate experiences using these UI "atoms," potentially reducing the need for traditional websites and ushering in a new era of agent-first application distribution.

## Detailed Notes

### The Problem with Text-Based Chat
- **Sub-optimal Experience:** Text-based chat responses are described as "really bad" and "sub-optimal" for complex data and interactions.
- **Loss of Identity:** Companies were hesitant to send data to platforms like ChatGPT because their tools would be reduced to a "wall of text," losing their branding and identity (e.g., Shopify, Booking, Expedia).
- **Lack of Interactivity:** Text responses offer no interactive elements, limiting user engagement and functionality.

### Evolution and Standardization of MCP Apps
- **MCPUI Origin:** Ido Salomon released MCPUI in May last year, introducing the concept of passing UI over MCP and enabling communication between the UI and the host.
- **Motivation:** Preserve existing UI/UX knowledge, maintain branding and identity, and adapt to the new agentic world.
- **Standardization:** MCPUI partnered with Anthropic and OpenAI to become MCP Apps, the first official extension to the MCP standard.
- **Early Adoption:** Even before standardization, companies like Shopify (millions of online stores sending MCPUI chunks) and Hugging Face (all spaces as MCPUI widgets) were early adopters.

### Industry Adoption and Community Engagement
- **Host Support:** MCP Apps are now supported by a wide range of hosts including VS Code, Cursor, Claude, ChatGPT, Microsoft Copilot, GitHub, Postman, Goose, and even Spy (a terminal application).
- **Official Recommendation:** ChatGPT recommends MCP Apps as the preferred way to build ChatGPT apps.
- **Community Growth:** Strong community adoption with plugins, workshops, advocates, and companies built around MCP Apps.
- **Work Group:** An official MCP Apps repo exists with Anthropic and OpenAI, featuring public work group meetings held tri-weekly to push the standard forward.
- **Vision:** To become the "global standard for UI inside chat apps."

### Core Concept: Passing Interactive UI over MCP
- **Resource Return:** Instead of a text response, an MCP server can return a UI resource (e.g., HTML) to the host.
- **Host Transformation:** The host, if it supports MCP Apps, takes the HTML and transforms it into an interactive application.
- **Interactivity:** The UI is not just presentational; it can communicate with the user and its backend.
- **Standardized Message Passing:** UI chunks send messages back to the host (e.g., a tool call). The host decides the next action, keeping everything in context and maintaining host control.
- **Practical Demo (Claude & PostHog):**
    - **Old Way:** Asking Claude to "analyze my funnel" would yield a long, textual response from PostHog, which is accurate but hard to digest.
    - **New Way:** With MCP Apps, asking "show me" results in a rich UI visualization directly from PostHog, preserving their branding and experience.
    - **Interactive Exploration:** Users can click on elements within the PostHog UI (e.g., a specific funnel step) to get follow-up information, demonstrating bidirectional interaction.

### Technical Architecture and Flow
1.  **User Prompt:** User prompts the host (e.g., "funnel data").
2.  **Tool Call:** Host sends a tool call to the MCP server.
3.  **Resource Return:** MCP server returns a UI resource (e.g., HTML) instead of text. This is implemented by registering a resource and returning it.
4.  **Host Rendering:** The host, supporting MCP Apps, takes the resource and renders it securely inside a sandbox. This involves a React component that accepts the UI resource and a callback for messaging.
5.  **User Interaction:** User interacts with the rendered UI (e.g., clicks a button).
6.  **Event Back to Model:** Events from the UI view are sent back to the model, which can trigger further tool calls, follow-up messages, or additional resource fetches, completing an end-to-end bidirectional flow.

### A New Era of Web Interaction
- **Beyond Websites:** This architecture ushers in a "new web" where traditional websites and numerous browser tabs may become less necessary.
- **Agentic Decomposition:** Agents can decompose complex UIs into "atoms" and build relevant experiences for the user.
- **Personalized Orchestration:** A proactive personal assistant can orchestrate tasks by pulling relevant UI chunks from different services (e.g., Google Calendar for an anniversary, Amazon for products, Booking for venues and maps based on user preferences).
- **Win-Win-Win:**
    - **Companies:** Maintain identity and branding (e.g., Google Calendar UI).
    - **Users:** Familiar interfaces, personalized experiences, all within one chat.
    - **Hosts:** Don't need to generate complex UIs, leveraging domain expertise from service providers.

### Interaction Mindset and Message Spectrum
- **Host-Controlled Journey:** In this new paradigm, the apps/services no longer directly own the user's journey; the host mediates interactions.
- **Message Spectrum:** MCP Apps standardize message types between the UI and the host, representing how much control the UI retains versus how much it delegates to the host:
    - **Notification:** UI notifies the host but handles the action itself (e.g., increasing items in a cart, action goes to Shopify, host is informed). Highest UI control.
    - **Tool Call:** UI instructs the host to call a specific tool.
    - **Prompt:** UI releases control, instructing the host to run a prompt. Lowest UI control.
- **Future Prediction:** Browsers and websites as we know them might be replaced by personal assistants accepting small UI chunks within two years. The goal is to standardize MCP Apps as a global UI standard by 2026.

### Future Outlook and Development
- **Rapid Evolution:** The MCP Apps spec is continuously evolving, with many updates shipped recently based on community feedback.
- **Official SDK (XApps):** Provides a compliant way for developers to build applications, including built-in skills for agent-driven app generation.
- **Reusable Views:** Future work includes enabling reusable views to improve performance for heavy applications (e.g., Autodesk) by referencing existing views and pushing data, rather than re-rendering.
- **Model-UI Interaction:** Developing a standardized way for models (e.g., Claude) to interact with the UI (e.g., clicking buttons, filling forms) by exposing tools from the app to the model.
- **Interoperability:** Working towards interoperability with other UI protocols like Google's A2UI (generative UI protocol) and WebMCP to build a unified standard.

### UI Generation Spectrum
- MCP Apps are agnostic to how the UI is generated, supporting a full spectrum:
    - **Predefined UI (Classic MCP App):**
        - Example: Airbnb building and sending its own "black box" UI.
        - Good for 80% of use cases.
    - **Declarative UI:**
        - App declares the UI structure (e.g., JSON render), but the host renders the components.
        - Host and app share UI functionality and visibility.
        - Allows hosts (e.g., Claude) to maintain a consistent look and feel across different apps.
    - **Fully Generative UI:**
        - Model generates the UI "out of thin air" (e.g., Claude's recent generative UI feature).
        - This generative UI is streamed into an MCP App, utilizing the protocol "under the hood."
        - Applicable for both third-party and first-party UI.

### Distribution and Getting Started
- **New Distribution Model:** MCP Apps represent a new way to distribute applications, leveraging the massive user bases of agent platforms.
- **Massive Audience:** ChatGPT alone has over 1 billion weekly users, representing a potential audience 160 times larger than the iPhone App Store at its launch.
- **For App Developers (Server):** Utilize the XApps repo and SDK, which includes skills for easy app creation.
- **For Host Developers (Client):** Use the MCPUI SDK (a React component) to enable out-of-the-box support for hundreds of MCP Apps.
- **Community Involvement:** Developers are encouraged to join the official MCP Apps repo (XApps), Discord channels (committee and community), and participate in discussions to shape the future of UI in agents.
- **Opportunity:** This is presented as a "once in a 20 years opportunity" to rethink applications, moving from monolithic apps to interconnected UI chunks within a smart model. An app written once can run everywhere.

## Key Technical Concepts

- **MCP Apps as UI Delivery Protocol:** The fundamental shift from MCP as a tool-calling protocol to a mechanism for delivering rich, interactive UI components (e.g., HTML resources) directly into agent hosts.
- **Sandboxed UI Rendering:** Hosts render the received UI within a secure, isolated environment to prevent malicious code execution and ensure stability.
- **Bidirectional Message Passing:** A standardized communication channel enabling the rendered UI to send events and messages back to the host/model, which can then trigger further actions, tool calls, or context updates.
- **Resource-Based UI Return:** Instead of returning plain text, tool calls made to an MCP server can now return a specific UI resource (like an HTML snippet or a declarative UI definition) that the host interprets and renders.
- **Interaction Message Spectrum:** A defined hierarchy of message types (Notification, Tool Call, Prompt) that allows the UI to specify the desired level of control and interaction with the host, from simply informing to fully delegating a prompt.
- **Generative UI Agnosticism:** The MCP Apps protocol is designed to be independent of the UI generation method, supporting predefined, declarative, and fully generative UIs, acting as the underlying transport layer for all.

## Practical Applications

- **Enhanced User Experience:** Deliver rich, interactive, and branded user interfaces directly within conversational AI environments, moving beyond generic text responses.
- **Agent-First Distribution:** Position products and services for massive reach by making them natively accessible and interactive within popular agent platforms like ChatGPT, Claude, and Copilot.
- **Preserve Brand Identity:** Companies can maintain their unique branding, design, and user experience within agent interactions, preventing their tools from being reduced to undifferentiated text.
- **Streamlined Workflows:** Enable users to complete complex, multi-service tasks (e.g., planning an event, booking travel) entirely within a single chat interface, with the agent orchestrating UI chunks from various providers.
- **Developer Efficiency:** Leverage official SDKs (XApps for app developers, MCPUI for host developers) to quickly build compliant applications and integrate interactive UI, ensuring compatibility across hosts.
- **Future-Proofing:** Adapt to the evolving web landscape where personal agents and modular UI components may increasingly replace traditional monolithic websites and browser-centric interactions.

## Notable Quotes

> "text is really bad and actually this was one of the main blockers of companies or tools not to send their data to ChatGPT because they didn't want to be reduced to like this thing, this wall of text where you don't have identity."

> "this is ushering a new web, a web where we don't need websites. We don't need all of those tabs just to organize an anniversary. We don't need to familiarize ourselves with bunch of different UIs."

> "I kind of hope that you don't look at this as a threat but more as an opportunity like basically a once in a 20 years opportunity to think about your apps again and think what is the core user experience that we're looking to get and imagine it not as a monolithic single app where people go to but actually a part of a new web of applications, these chunks of UI that allow you to communicate between each other using a smart model in between."

## Timestamps

| Time | Topic |
|------|-------|
| 0:00 | Introduction to MCP Apps |
| 1:02 | Why we need MCP Apps: Moving beyond text-based chat |
| 2:06 | Evolution of the MCP UI standard and partnerships |
| 3:25 | Industry and community adoption |
| 5:14 | Core concepts: Passing UI over MCP |
| 6:49 | Practical demonstration (PostHog and Claude) |
| 8:54 | Technical architecture: How it works |
| 10:23 | A new era of web interaction and user experience |
| 12:32 | Interaction mindset and message spectrums |
| 14:56 | Future outlook: Reusable views and model-UI interaction |
| 16:18 | Spectrum of UI generation: Predefined vs. Generative |

## Takeaways

- MCP is evolving from a tool-calling protocol to a UI delivery protocol
- The predefined-to-generative UI spectrum gives developers flexibility
- Early adoption of MCP UI standards positions products for agent-first distribution
- The interaction model is fundamentally different from traditional web apps

## Speaker Links

- Liad Yosef and Ido Salomon via MCP Apps
