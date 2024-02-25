Resource - https://www.youtube.com/playlist?list=PLOspHqNVtKAAsiohuZj1Bt4XpA3_bkS3c


### LLM 
large datasets - text/text like things - Foundation Model - Petabytes of data - more parameter more complex 
1. Data - enormous amount of text data
2. Architecture - Neural Network (Transformer specifically for GPT)
3. Training - predicts 
Fine tuning converts the General Model into Specific task capable model

Business Applications -> Conversational ChatBots, Content Creation, Software development.

![[LLMs.png]]

### Fine Tuning vs Prompt Tuning
FT - Improve performance of pre-trained model - gather and label example of target task so as to fine tune the model
PT - limited dataset - best frontend cues, prompts are fed to AI model to give context of the task
PT -> Prompt Engineering - guide a llm to a specific task
Large number of prompts required to engineer a generic model to specific task so we need SOFT PROMPT - which are AI engineered prompts rather than Hard Prompts (Human made)
![[Types of tuning.png]]
Inshort, 
1. FT - provide tuners to the pre trained model then provide inputs, 
2. PE - provide extra engineered prompts to the pre trained model before providing the inputs for a specific task
3. PT - provide soft prompts before the inputs to 

### Risks
1. Hallucinations - incorrect answers based on wrong facts as it is predicts it
2. Bias - reclining to a particular fashion based on training
3. Consent
4. Security - Jail breaking, malicious intents

Mitigations Strategies
1. Explainability
2. Culture and Audits
3. Accountability - complaint and governance policies
4. Education - 
