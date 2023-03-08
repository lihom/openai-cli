# OpenAI on python

## Env require

1. python 3.10 or above
```
mac: https://formulae.brew.sh/formula/python@3.10
linux: https://computingforgeeks.com/how-to-install-python-on-debian-linux/
```

2. pip install (refs: https://www.maxlist.xyz/2019/07/13/pip-install-python/)
```
curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
python3.10 get-pip.py
rm get-pip.py
```

## Install

```
pip install openai
```

## Prepare API Key

[Video Tutorial] (https://www.youtube.com/watch?v=nafDyRsVnXU)

## Create .py file put code into file

``` bash
vi gpt.py
```

``` python
import os
import openai
import sys
import json
import csv
import re

class myGpt:
  debug = False
  apikey = '<YOUR_API_KEY>'

  question = "describe ChatGPT openai api and show python example."
  answer_length = 255
  temperature = 1
  model = 'text-davinci-003'

  _argsObj = {}

  def __init__(self):
    self.parseArg()

    self.question = self.getArg('question', self.question)
    self.answer_length = self.getArg('answer_length', self.answer_length)
    self.temperature = self.getArg('temperature', self.temperature)
    self.model = self.getArg('model', self.model)
    
    act = self.getArg('act')

    if act:
      self.parseActPrompt(act)

    self.runAPI()

  def runAPI(self):
    if self.debug:
      print(self.question)
      print(self.answer_length)
      print(self.temperature)
      print(self.model)
      exit()

    # Load your API key from an environment variable or secret management service.
    openai.api_key = self.apikey

    if self.model == 'gpt-3.5-turbo' or self.model == 'gpt-3.5-turbo-0301':
      response = openai.ChatCompletion.create(
        model = self.model,
        messages = [{
          "role": "user",
          "content": self.question
        }],
        temperature = self.temperature,
        max_tokens = self.answer_length
      )
    else:
      response = openai.Completion.create(
        model = self.model,
        prompt = self.question,
        temperature = self.temperature,
        max_tokens = self.answer_length
      )

    # Convert response to string and decode unicode.
    response_str = json.dumps(response, ensure_ascii = False)

    # Load json string to data.
    response_data = json.loads(response_str)

    # Decode escape string.
    if self.model == 'gpt-3.5-turbo' or self.model == 'gpt-3.5-turbo-0301':
      result = response_data['choices'][0]['message']['content'].replace('\\n', '\n')
    else:
      result = response_data['choices'][0]['text'].replace('\\n', '\n')

    print(result)

  def parseArg(self):
    args = sys.argv

    for i, v in enumerate(args):
      if i == 0:
        continue
      
      arg = args[i].split('=')
      if (len(arg) == 2):
        self._argsObj[arg[0]] = arg[1]

  def getArg(self, name, default_value = False):
    args = sys.argv

    if name == 'question':
      if '--question' in self._argsObj:
        return self._argsObj['--question']
      elif args and len(args) >= 2 and args[1].find('--') == -1:
        return args[1]

    elif name == 'answer_length':
      if 'answer_length' in self._argsObj:
        return int(self._argsObj['--answer_length'])
      elif args and len(args) >= 3 and args[2].find('--') == -1:
        return int(args[2])

    elif name == 'temperature':
      if '--temperature' in self._argsObj:
        return int(self._argsObj['--temperature'])

    elif name == 'act':
      if '--act' in self._argsObj:
        return self._argsObj['--act']

    elif name == 'model':
      if '--model' in self._argsObj:
        if self._argsObj['--model'] == 'gpt-3.5-turbo' or self._argsObj['--model'] == 'gpt-3.5-turbo-0301':
          print("notice: when model use [" + self._argsObj['--model'] + "] openai library version require 0.36.0 or above.")
          print("\n")
        
        return self._argsObj['--model']

    return default_value

  def parseActPrompt(self, act):
    with open('../prompts.csv', newline = '') as csvfile:
      rows = csv.DictReader(csvfile)

      for row in rows:
        if act == row['act']:
          re_match = re.match("^(.*)\"(.*)\"$", row['prompt'])
          if re_match:
            self.question = re.sub("\"(.*)\"", '"' + self.question + '"', row['prompt'])
          else:
            self.question = row['prompt'] + ' "' + self.question + '"'
          break

myGpt()
```

``` bash
vi dall.py
```

``` python
import os
import openai
import sys
import json

class myDall:
  debug = False
  apikey = '<YOUR_API_KEY>'

  question = "The Shiba Inu and the cat sat on the ground and drew in the style of Peter Rabbit's illustrations."
  images_number = 1
  image_size = '512x512'

  _argsObj = {}

  def __init__(self):
    self.parseArg()

    self.question = self.getArg('question', self.question)
    self.images_number = self.getArg('images_number', self.images_number)
    self.image_size = self.getArg('image_size', self.image_size)

    self.runAPI()

  def runAPI(self):
    if self.debug:
      print(self.question)
      print(self.images_number)
      print(self.image_size)
      exit()

    # Load your API key from an environment variable or secret management service.
    openai.api_key = self.apikey

    response = openai.Image.create(
      prompt = self.question,
      n = self.images_number,
      size = self.image_size
    )

    # Convert response to string and decode unicode.
    response_str = json.dumps(response)

    # Load json string to data.
    response_data = json.loads(response_str)

    # Foreach output result.
    print('\n')

    for item in response_data['data']:
      print(item['url'] + '\n')

  def parseArg(self):
    args = sys.argv

    for i, v in enumerate(args):
      if i == 0:
        continue
      
      arg = args[i].split('=')
      if (len(arg) == 2):
        self._argsObj[arg[0]] = arg[1]

  def getArg(self, name, default_value = False):
    args = sys.argv

    if name == 'question':
      if '--question' in self._argsObj:
        return self._argsObj['--question']
      elif args and len(args) >= 2 and args[1].find('--') == -1:
        return args[1]

    elif name == 'images_number':
      if 'images_number' in self._argsObj:
        return int(self._argsObj['--images_number'])
      elif args and len(args) >= 3 and args[2].find('--') == -1:
        return int(args[2])

    elif name == 'image_size':
      if '--image_size' in self._argsObj:
        if self._argsObj['--image_size'] not in ['256x256', '512x512', '1024x1024']:
          print("note: DALL.E only support 256x256, 512x512, 1024x1024 three size.")
          exit()

        return self._argsObj['--image_size']

    return default_value

myDall()
```

## Run openai on terminal

``` bash
# Ask question get answer length 255.
python3.10 gpt.py "Describe your self." 255
```

``` bash
# Use act prompts.
python3.10 gpt.py "How important grid system please describe it." 1000 --act="Web Design Consultant"
```

``` bash
# Use model.
python3.10 gpt.py "How important grid system please describe it." 1000 --model="gpt-3.5-turbo"
```

``` bash
# DALL.E only support 256x256, 512x512, 1024x1024 three size.
python3.10 dall.py "A openAI self-portrait." 1 --image_size="256x256"
```

* tips:
Your env property not have python cli, you can check /usr/bin folder, find your python3.10 bin file, in my case my python3.10 in /usr/local/bin/python3.10.

``` bash
# If you want to use python in command run those command. (or use python3.10 gpt.py)

sudo ln -s /usr/local/bin/python3.10 /usr/bin/python
```
