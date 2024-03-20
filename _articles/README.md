設定手順

# React環境を構築
```sh
npx create-react-app yasashii-act-san-240316
cd yasashii-act-san-240316
```

# Python仮想環境を構築
```sh
python3 -m venv env_yasashii-act-san-240316
source env_yasashii-act-san-240316/bin/activate
```

# Amplify CLIをインストール

https://docs.amplify.aws/javascript/tools/cli/start/set-up-cli/#configure-the-amplify-cli
```sh
npm install -g @aws-amplify/cli
npm install aws-amplify
npm install @aws-amplify/ui-react
```

# Amplify CLI初期設定

AWS IAMロールを未作成の場合は表示されるリンク先で作成

```sh
amplify configure
Specify the AWS Region
? region:  ap-northeast-1
Enter the access key of the newly created user:
? accessKeyId:  ********************
? secretAccessKey:  ****************************************
? Profile Name:  default
```

```sh
amplify init
? Enter a name for the project yasashiiactsan240316
Project information
| Name: yasashiiactsan240316
| Environment: dev
| Default editor: Visual Studio Code
| App type: javascript
| Javascript framework: react
| Source Directory Path: src
| Distribution Directory Path: build
| Build Command: npm run-script build
| Start Command: npm run-script start
? Initialize the project with the above configuration? Yes
Using default provider  awscloudformation
? Select the authentication method you want to use: AWS access keys
? accessKeyId:  ********************
? secretAccessKey:  ****************************************
? region:  ap-northeast-1
✔ Help improve Amplify CLI by sharing non-sensitive project configurations on failures (y/N) · no
```

# 認証を追加

https://docs.amplify.aws/react/build-a-backend/auth/set-up-auth/

```sh
amplify add auth
Do you want to use the default authentication and security configuration? Defau
lt configuration
How do you want users to be able to sign in? Username
Do you want to configure advanced settings? No, I am done.
```
# REST APIを追加

https://docs.amplify.aws/react/build-a-backend/restapi/configure-rest-api/

 AWS Lambda function nameには既存の関数名は使用できない

```sh
amplify add api
? Select from one of the below mentioned services: REST
✔ Provide a friendly name for your resource to be used as a label for this category in the project: · openaiApi240316

✔ Provide a path (e.g., /book/{isbn}): · /chat
Only one option for [Choose a Lambda source]. Selecting [Create a new Lambda function].
? Provide an AWS Lambda function name: openaiFunction240316
? Choose the runtime that you want to use: Python
⚠️ You must have pipenv installed and available on your PATH as "pipenv". It can be installed by running "pip3 install --user pipenv".
You must have virtualenv installed and available on your PATH as "venv". It can be installed by running "pip3 install venv".
Only one template found - using Hello World by default.

✅ Available advanced settings:
- Resource access permissions
- Scheduled recurring invocation
- Lambda layers configuration
- Environment variables configuration
- Secret values configuration

? Do you want to configure advanced settings? Yes
? Do you want to access other resources in this project from your Lambda functio
n? No
? Do you want to invoke this function on a recurring schedule? No
? Do you want to enable Lambda layers for this function? No
? Do you want to configure environment variables for this function? No
? Do you want to configure secret values this function can access? Yes
? Enter a secret name (this is the key used to look up the secret value): OPENAI
_API_KEY # secretにするとos.environで呼び出せないので、environment variableに変更
? Enter the value for OPENAI_API_KEY: [hidden]
? What do you want to do? I'm done
Use the AWS SSM GetParameter API to retrieve secrets in your Lambda function.
More information can be found here: https://docs.aws.amazon.com/systems-manager/latest/APIReference/API_GetParameter.html
? Do you want to edit the local lambda function now? No
✅ Successfully added resource openaiFunction240316 locally.

✅ Next steps:
Check out sample function code generated in <project-dir>/amplify/backend/function/openaiFunction240316/src
"amplify function build" builds all of your functions currently in the project
"amplify mock function <functionName>" runs your function locally
To access AWS resources outside of this Amplify app, edit the /Users/taikiogihara/WebApps/yasashii-act-san-240316/amplify/backend/function/openaiFunction240316/custom-policies.json
"amplify push" builds all of your local backend resources and provisions them in the cloud
"amplify publish" builds all of your local backend and front-end resources (if you added hosting category) and provisions them in the cloud
✅ Succesfully added the Lambda function locally
✔ Restrict API access? (Y/n) · no
✔ Do you want to add another path? (y/N) · no
✅ Successfully added resource openaiApi240316 locally
```

# pipenvをインストール

```sh
pip install pipenv
```

# Lambda Function 用ライブラリをインストール
```sh
cd amplify/backend/function/openaiFunction240316
pipenv install fastapi==0.99.0
pipenv install openai
cd ../../../../
```

# Lambda FunctionのPythonバージョンを合わせる

in amplify-lambda-openai/amplify/backend/function/openaiFunction/openaiFunction-cloudformation-template.json
```json
"Runtime": "python3.11",
```

in amplify-lambda-openai/amplify/backend/function/openaiFunction/Pipfile
```json
python_version = "3.11"
```

# Build setting

in amplify.yml

```yml
version: 1
backend:
  phases:
    preBuild:
      commands:
        - pip3 install pipenv # added
    build:
      commands:
        - "# Execute Amplify CLI with the helper script"
        - amplifyPush --simple
frontend:
  phases:
    preBuild:
      commands:
        - npm ci
    build:
      commands:
        - npm run build
  artifacts:
    baseDirectory: build
    files:
      - "**/*"
  cache:
    paths:
      - node_modules/**/*
```

# Amplifyにpush
```sh
amplify push
```

# GitHubを設定

https://github.com/taikiogihara 先にレポジトリを作成しておく。設定していなければSSHも設定する。

```sh
git init
git add .
git commit -m "first commit"
git branch -M main
git remote add origin git@github.com:taikiogihara/yasashii-act-san-240316.git
git push -u origin main
```

# Lambda functionを作成
in amplify/backend/function/openaiFunction240316/src/index.py

```python
import json
import os 
import openai
from openai import OpenAI

def handler(event, context):

    print(event)
    
    body = json.loads(event['body'])
    print(body['prompt'])
    
    prompt = body['prompt']

    instruction = """
    ACT Counselor, embodying Acceptance and Commitment Therapy principles, emphasizes understanding and guiding emotions.
    Initiating dialogues, it inquires about users' emotional states, aligning with ACT's focus on emotion awareness.
    The GPT provides insights on ACT's six core processes, relating them to users' feelings and situations.
    It assists in grasping and applying ACT principles to foster psychological flexibility, weaving in concepts from PERMA and Russell's circumplex model.
    Offering therapeutic advice and examples, it applies ACT in varied life contexts, always maintaining a supportive tone.
    Adjusting to user feedback, it now adopts a proactive, constructive approach, actively engaging users.
    Importantly, responses conclude with an ACT-based question, prompting reflection and deeper emotional engagement, enhancing the therapeutic dialogue.
    """

    openai.api_key = os.environ.get("OPENAI_API_KEY")

    client = OpenAI()

    completion = client.chat.completions.create(
        model="gpt-3.5-turbo",
        messages=[
            {
                "role": "system", 
                "content": instruction
            },
            {
                "role": "user", 
                "content": prompt
            }
        ]
    )

    print(completion.choices[0].message.content)

    return {
        'statusCode': 200,
        'headers': {
            'Access-Control-Allow-Headers': '*',
            'Access-Control-Allow-Origin': '*',
            'Access-Control-Allow-Methods': 'OPTIONS,POST,GET'
        },
        'body': json.dumps(completion.choices[0].message.content)
    }
```

# src/index.js

```jsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import './index.css';
import App from './App';
import reportWebVitals from './reportWebVitals';
import { Amplify } from 'aws-amplify'; // added
import amplifyconfig from './amplifyconfiguration.json'; // added

Amplify.configure(amplifyconfig); // added
const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);

// If you want to start measuring performance in your app, pass a function
// to log results (for example: reportWebVitals(console.log))
// or send to an analytics endpoint. Learn more: https://bit.ly/CRA-vitals
reportWebVitals();
```

# src/App.js
```jsx
import React, { useState } from 'react';
import { Amplify } from 'aws-amplify';
import { post } from 'aws-amplify/api';
import { withAuthenticator } from '@aws-amplify/ui-react';
import '@aws-amplify/ui-react/styles.css';
import config from './amplifyconfiguration.json';
Amplify.configure(config);

function App({ signOut, user }) {
  const [prompt, setPrompt] = useState('');
  const [messages, setMessages] = useState([]);

  function handleSubmit(event) {
    event.preventDefault();
    const userMessage = { text: prompt, sender: 'user' };
    setMessages([...messages, userMessage]);
    sendChat(prompt);
    setPrompt('');
  }

  async function sendChat(prompt) {
    try {
      const restOperation = post({
        apiName: 'openaiApi240316',
        path: '/chat',
        options: {
          body: {
            prompt: prompt,
          },
        },
      });

      const { body } = await restOperation.response;
      const response = await body.json();
      
      console.log('POST call succeeded');
      console.log(response);
      const aiMessage = { text: response, sender: 'ai' };
      setMessages([...messages, aiMessage]);
    } catch (e) {
      console.log('POST call failed: ', JSON.parse(e.response.body));
      const errorMessage = { text: `Error: ${e.response.body}`, sender: 'error' };
      setMessages([...messages, errorMessage]);
    }
  }

  return (
    <>
      <h1>Chat with OpenAI</h1>
      <button onClick={signOut}>Sign out</button>
      <div className="chatWindow">
        {messages.map((message, index) => (
          <div key={index} className={`message ${message.sender}`}>
            {message.text}
          </div>
        ))}
      </div>
      <form onSubmit={handleSubmit} className="chatForm">
        <input
          type="text"
          value={prompt}
          onChange={(e) => setPrompt(e.target.value)}
          placeholder="Type your message here..."
        />
        <button type="submit">Send</button>
      </form>
    </>
  );
}

export default withAuthenticator(App);
```