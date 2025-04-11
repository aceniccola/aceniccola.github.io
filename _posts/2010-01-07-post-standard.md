---
title: "Stanford 5th LLM x LAW Hackathon"
excerpt_separator: "<!--more-->"
categories:
  - AI
tags:
  - Hackathons
---
In this hackathon, Aidan and I created an application to help lawers distinguish between a valid and invaild response to an argument. We used a series of agents and tools to summarerize, analyze and verify the possible responses to each argument. 

The first step is summarization. We created a summarizer agent that went through each argument and potential counterargument of the whole dataset and summarized them each. The summary is in the following form:

```python
from pydantic import BaseModel, Field
class Summary(BaseModel):
    """Summary to assist in decision making regarding a specific argument."""
    what: str = Field(description="A concise summary of the argument's core message. What is it really saying?", max_length=150)
    who: str = Field(description="Identify the speaker (or author) and their likely intended audience.", max_length=100)
    why: str = Field(description="Infer the likely motive or goal behind presenting this argument.", max_length=100)
    what_to_do: str = Field(description="Suggest the single most effective way to respond to or counter this argument.", max_length=150)

```

This summary class is then incorperated into the list of tools the summarizer model has access to: ``` tools.append(Summary) ```

Then we just check if the model outputs a Summary object. If so we have completed the task, otherwise, the llm needs to be fed back the response that was outputed from the tool it did choose for it to decide best how to summarize the argument in the form requested. 

The next step is to analyze the arguments in the context of their possible responses. This step can be combined with the verification step in a single graph (as, if a response seems to fit an argument, it must be verified and if it must be verified then, it must have been choosen as having seemed to fit an argument).

Therefore we have an analyser node that passes it on to either a tool node or, if it has decided on the responses that most aptly fit, passes it to the verification node which looks at each response more strictly and sends it back to the origional agent with feedback if it finds a descrepency. The key is to make sure that both agents see all the summaries informed by the summarizer and that the first llm tends to initially overinclude responses and the verifier is overly critical. Then the inital agent and the verifier will eventually come to a conscensis. With both of them sending back their rationalities for why they disagree with each other, they can better approach the actual solution. 

I learned in this hackathon that, not only is it 
