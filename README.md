# OpenAI CLI

Create OpenAI CLI tool on Python and NodeJS.

## Special Thanks

Thanks for [Awesome ChatGPT Prompts](https://github.com/f/awesome-chatgpt-prompts) Support Act Prompts.

## Getting Started

1. Go to python or node folder.
2. Read README.md
3. Install software and openai library.
4. Create code.
5. Run code on terminal.

## Example

### Python

``` bash
# GPT
python3.10 gpt.py "How important grid system please describe it." 1000 --act="Web Design Consultant" --model="gpt-3.5-turbo"

# DALL.E
python3.10 dall.py "A openAI self-portrait." 1 --image_size="256x256"
```

### NodeJS

``` bash
# GPT
node gpt.js "How important grid system please describe it." 1000 --act="Web Design Consultant" --model="gpt-3.5-turbo"

# DALL.E
node dall.js "A openAI self-portrait." 1 --image_size="256x256"
```

## Help

### GPT

``` bash
node gpt.js "[question]" [answer length]
```

--act
```
Act Prompts form Awesome ChatGPT Prompts
```
[readmore](https://github.com/f/awesome-chatgpt-prompts/blob/main/prompts.csv)

--model
```
text-davinci-003
text-curie-001
text-babbage-001
text-ada-001
code-davinci-002
code-cushman-001
gpt-3.5-turbo
gpt-3.5-turbo-0301
gpt-4
gpt-4-0314
gpt-4-32k
gpt-4-32k-0314
```

--temperature
```
between 0 and 1
0.2: more focused
0.8: more random
```

### DALL.E

``` bash
node dall.js "[question]" [number of images]
```

--image_size
```
must be one of
256x256, 512x512, 1024x1024
```
