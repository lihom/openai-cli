# OpenAI on node.js

## Env require

1. node 8.x and 10.x
2. npm (version check: https://nodejs.org/zh-cn/download/releases/)

## Install

```
npm install openai
npm install csv
```

## Prepare API Key

[Video Tutorial] (https://www.youtube.com/watch?v=nafDyRsVnXU)

## Create .js file put code into file

``` bash
vi gpt.js
```

``` js
const { Configuration, OpenAIApi } = require("openai");
const fs = require("fs");
const csv = require("csv");

class myGpt {
  debug = false;
  apikey = '<YOUR_API_KEY>'

  question = "describe ChatGPT openai api and show python example.";
  answer_length = 255;
  temperature = 1;
  model = 'text-davinci-003';

  _argsObj = {};

  constructor()
  {
    this.parseArg();

    this.question = this.getArg('question', this.question);
    this.answer_length = this.getArg('answer_length', this.answer_length);
    this.temperature = this.getArg('temperature', this.temperature);
    this.model = this.getArg('model', this.model);
    
    let act = this.getArg('act');

    if (act)
    {
      this.parseActPrompt(act);
    }
    else
    {
      this.runAPI();
    }
  }

  async runAPI()
  {
    if (this.debug)
    {
      console.log(this.question);
      console.log(this.answer_length);
      console.log(this.temperature);
      console.log(this.model);
      return;
    }

    const configuration = new Configuration({
      apiKey: this.apikey
    });

    const openai = new OpenAIApi(configuration);

    if (this.model == 'gpt-3.5-turbo' || this.model == 'gpt-3.5-turbo-0301')
    {
      const response = await openai.createChatCompletion({
        model: this.model,
        messages: [{
          role: 'user',
          content: this.question
        }],
        temperature: this.temperature,
        max_tokens: this.answer_length
      });

      console.log(response.data.choices[0].message.content);
    }
    else
    {
      const response = await openai.createCompletion({
        model: this.model,
        prompt: this.question,
        temperature: this.temperature,
        max_tokens: this.answer_length
      });

      console.log(response.data.choices[0].text);
    }
  }

  parseArg()
  {
    let args = process.argv;

    for (let i = 0; i < args.length; i++)
    {
      if (i < 2) continue;

      let arg = args[i].split('=');
      if (arg.length == 2) this._argsObj[arg[0]] = arg[1];
    }
  }

  getArg(name, default_value = false)
  {
    let args = process.argv;

    if (name == 'question')
    {
      if (this._argsObj['--question'] != undefined)
      {
        return this._argsObj['--question'];
      }
      else if (args.length >= 3 && args[2].indexOf('--') == -1)
      {
        return args[2]
      }
    }
    else if (name == 'answer_length')
    {
      if (this._argsObj['--answer_length'] != undefined)
      {
        return Number(this._argsObj['--answer_length']);
      }
      else if (args.length >= 4 && args[3].indexOf('--') == -1)
      {
        return Number(args[3])
      }
    }
    else if (name == 'temperature')
    {
      if (this._argsObj['--temperature'] != undefined)
      {
        return this._argsObj['--temperature'];
      }
    }
    else if (name == 'act')
    {
      if (this._argsObj['--act'] != undefined)
      {
        return this._argsObj['--act'];
      }
    }
    else if (name == 'model')
    {
      if (this._argsObj['--model'] != undefined)
      {
        if (this._argsObj['--model'] == 'gpt-3.5-turbo' || this._argsObj['--model'] == 'gpt-3.5-turbo-0301')
        {
          console.log("\x1b[36m", "notice: when model use [" + this._argsObj['--model'] + "] openai library version require 3.2.1 or above");
          console.log("\x1b[0m", "\n");
        }

        return this._argsObj['--model'];
      }
    }

    return default_value;
  }

  parseActPrompt(act)
  {
    let rows = [];
    let keys = [];

    fs.createReadStream('../prompts.csv')
      .pipe(csv.parse())
      .on('data', (data) => {
        if (keys.length == 0)
        {
          for (let k in data) keys.push(data[k]);
        }
        else
        {
          let obj = {};
          for (let k in keys) obj[keys[k]] = data[k];
          rows.push(obj);
        }
      })
      .on('end', () => {
        for (let k in rows)
        {
          let row = rows[k];
          if (act == row.act)
          {
            let re_match = row.prompt.match(/^(.*)\"(.*)\"$/);
            if (re_match)
            {
              this.question = row.prompt.replace(/\"(.*)\"/, '"' + this.question + '"');
            }
            else
            {
              this.question = row.prompt + ' "' + this.question + '"';
            }
            break;
          }
        }

        this.runAPI();
      });
  }
}

new myGpt();
```

``` bash
vi dall.js
```

``` js
const { Configuration, OpenAIApi } = require("openai");

class myDall {
  debug = false;
  apikey = '<YOUR_API_KEY>'

  question = "The Shiba Inu and the cat sat on the ground and drew in the style of Peter Rabbit's illustrations.";
  images_number = 1;
  image_size = '512x512';

  _argsObj = {};

  constructor()
  {
    this.parseArg();

    this.question = this.getArg('question', this.question);
    this.images_number = this.getArg('images_number', this.images_number);
    this.image_size = this.getArg('image_size', this.image_size);
    
    this.runAPI();
  }

  async runAPI()
  {
    if (this.debug)
    {
      console.log(this.question);
      console.log(this.images_number);
      console.log(this.image_size);
      return;
    }

    const configuration = new Configuration({
      apiKey: this.apikey
    });

    const openai = new OpenAIApi(configuration);

    const response = await openai.createImage({
      prompt: this.question,
      n: this.images_number,
      size: this.image_size
    });

    console.log("\n");

    for (var k in response.data.data)
    {
      console.log(response.data.data[k].url + "\n");
    }
  }

  parseArg()
  {
    let args = process.argv;

    for (let i = 0; i < args.length; i++)
    {
      if (i < 2) continue;

      let arg = args[i].split('=');
      if (arg.length == 2) this._argsObj[arg[0]] = arg[1];
    }
  }

  getArg(name, default_value = false)
  {
    let args = process.argv;

    if (name == 'question')
    {
      if (this._argsObj['--question'] != undefined)
      {
        return this._argsObj['--question'];
      }
      else if (args.length >= 3 && args[2].indexOf('--') == -1)
      {
        return args[2]
      }
    }
    else if (name == 'images_number')
    {
      if (this._argsObj['--images_number'] != undefined)
      {
        return Number(this._argsObj['--images_number']);
      }
      else if (args.length >= 4 && args[3].indexOf('--') == -1)
      {
        return Number(args[3])
      }
    }
    else if (name == 'image_size')
    {
      if (this._argsObj['--image_size'] != undefined)
      {
        if (['256x256', '512x512', '1024x1024'].includes(this._argsObj['--image_size']) == false)
        {
          console.log("DALL.E only support 256x256, 512x512, 1024x1024 three size.");
          return;
        }

        return this._argsObj['--image_size'];
      }
    }

    return default_value;
  }
}

new myDall();
```

## Run openai on terminal

``` bash
# Ask question get answer length 255.
node gpt.js "Describe your self." 255
```

``` bash
# Use Act Prompts.
node gpt.js "How important grid system please describe it." 1000 "Web Design Consultant"
```

``` bash
# DALL.E only support 256x256, 512x512, 1024x1024 three size.
node dall.js "A openAI self-portrait." 1 "256x256"
```