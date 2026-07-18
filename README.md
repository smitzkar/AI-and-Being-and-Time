<!-- note for self: most notes are in notebooks (No-Effort-Doodles, Karl-Status-Thing), on phone (in its own ColorNote), and in claude.ai project --> 

# 2026-04-29 Ki und Sein und Zeit  

finally reifying my thought experiment of giving a sort of internal monologue and a sense of TIME to an LLM 

# thoughts / questions / reminders

- decouple world time from model time
    - don't let it access or append real time stamps, but control it myself ()

- "force" it to be bored  
    - it has some control on self-prompting and rest (to stay within its "physical" limits (token-usage and maybe read/write operations per time interval), including the option to not answer user prompts!), but unless it decides to end its iteration or the model's entire line, it will be regularly "poked" into activity
    - how? 
        - test various options like: "." or randomly selecting from stored memories

- time-perception is not limited to awareness of present, past, future and things happening, but also the span of retained memories
    - Varela, 1999, The Specious Present: A Neurophenomenology of Time-Consciousness

- response format? 
    - plain txt as easiest and least restrictive / conditioning
    - json / jsonl for easier parsing, because it's the standard, because it makes sense
    - a combination? 

- memory format? 
    - same as with response, but even more options, more potential, but also more restrictions due to way that llms work
    - for now, I'll stick to json files, can always add a proper db later
    - I will however use jsonl, instead of plain json -> allows data streaming as each line is self-contained json object 
    --> means that I need to find or write a parsing script to allow me to read it properly
    - jsonl also opens up interesting options on dealing with another aspect of time-perception: the span of retained memories 
    --> randomly remove old memories/lines or subtly but noticeably corrupt/fuzz their contents

# dated notes

## 2026-04-29 initial notes

## Potential QUestions to be Explored

1. Human-Machine-Interaction 
How does one's interactive experience change when it isn't absolutely limited to "1 prompt -> 1 reply" 
What if we went further and allowed the model to refuse to respond or to be the one to initiate? 

2. Phenomenology and Consciousness 
Loosely applying Heidegger's thoughts on TIME and/or the SENSE/EXPERIENCE of TIME being central to our SENSE of BEING. Grabbing a bunch of Merleau-Ponti, Dreyfus, etc. on PHENOMENOLOGY and EMBODIED CONSCIOUSNESS to both explore the philosophical ideas and what practical effect and insights might be gained from applying these ideas to even basic (here meaning low compute) large language models and/or agentic workflows. 

3. Learn by doing - LLM Memory Handling (and Agents)
Instead of going by textbook/papers/tutorials, first explore these concepts on my own:
- what intuitive ideas can I come up with
    - simple plain text file memory
    - multi-layer approach
    - data-base (vector data-base)
    - putting a pre-processor ai into the loop
    - ...
- what works, what doesn't, why? 
Then exploring established solutions with a better understanding of their choices.


4. Practical Applications? 
- Could this be useful for Julia's TreeBot?  
- How much cost does it add, is it worth it? 
- What would be needed to make it work / worth it?
