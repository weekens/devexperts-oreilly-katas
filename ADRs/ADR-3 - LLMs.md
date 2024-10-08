# **Status**

Approved

Ivan Kunyankin, Viktor Isaev, Denis Iudovich, Kiril Stoilov

# **Context**

The system’s advantage is twofolds. It ensures the level playing field for candidates and makes the hiring process 
easier for both candidates and employers. One of the key components in this solution is an LLM used for anonymising 
resumes, generating resume tips and extracting entities like person’s key responsibilities and obtained skills for 
matching.

In this document we justify the chosen approach for LLM usage.

There most likely will be several runs through an LLM. Here are the candidates:

1. Anonymisation
2. Anonymisation critic (different input, different prompt)
3. Resume tips generation
4. Entity extraction (extracts key responsibilities, skills and stack from resumes, summary that will be used for 
matching)

We need to choose a solution to use for LLMs.

# **Decision**

## AWS Bedrock

We could use one of the models from Anthropic (e.g., Claude 3.5) or Meta (e.g., llama 3.2) - they are ones of most 
advanced on the market. Their advanced reasoning capabilities will be beneficial for following a demanding instruction 
(such as extracting keywords/skills from job postings and ordering them by importance).

## Alternatives

- Other cloud providers (Azure, GCP). Cloud providers also compete with each other fiercely. Most of the AI-related 
services are present in each of them. However, as the overall system is built using AWS infrastructure, it  makes sense 
to stick with AWS as a fully managed service providing access to various LLMs.
- OpenAI API. Despite OpenAI leading the LLM world with their flagship models, sticking with them would mean limiting 
ourselves to a single model provider and more importantly, being have to send personal data outside of the system’s 
infrastructure, which weakens the security of our system (an important and one of the driving characteristics).
- Self-hosted open-source models. Open-source LLMs are becoming more capable. Despite open-source LLMs becoming more 
capable, costs saved from the model running would be eaten by the need to manage the infrastructure and the high 
resource demands of the LLMs.
- LLM alternatives. There are some other models and approaches capable of processing textual data. However, things we 
need to do require from the model advanced understanding of natural language which is a strong side of LLM models. 
Moreover, them being trained on enormous datasets means they are generalised enough to cover different domains. Other 
solutions however, would have been better in terms of hosting and managing costs.

# **Consequences**

## **Positives:**

- Holistic architecture - other parts of the system is on Amazon, which is simpler to understand and operate
- Access to models from different providers - increases flexibility and evolvability
- Fully-managed service - low operating costs
- Possibility to increase security by switching to a closed network
- Possibility to increase safety by using the *guardrails* feature
- The same service can be used for matching (embedding models)

## **Negatives:**

- Interpretability is not the strongest side of LLMs. They can fail silently and gradually degrade. This is hard to 
detect and can make an unprepared solution vulnerable. We try to mitigate these risks with introducing a critic model 
and monitoring. For the critical part of the process - anonymisation, for example, we add a Critic model. We can also 
monitor how often resumes are edited after the tips have been presented; how many entities on average our model 
extracts, etc.
- We need to make several runs for each resume which is costly. We can start optimising it by delegating simpler tasks 
to smaller models. Collecting data generated by a capable model could open up an opportunity to fine-tune a small model 
that would bring down the costs.

## Reversibility

This decision is easy to change. LLMs are stateless and basically are black boxes that receive an input and provide the 
output. We can change the box and pick another one, without a significant rework of the architecture.

