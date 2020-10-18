## Method 2: 
Using firebase functions.
  1. Either javascript
  1. Or typscript
  
 The overall method is same. The syntax to write code in typescript(index.ts) and javascript(index.js) may or may not vary.
 
 Requires gactions to have the command options: 
 
 * gactions update...

* gactions test...

NOTE: 
Object_Recogniser and cameraActions containers will be required in all methods by default. Current process is for building assistant container.

## Resources
1. [Google Assistant Quick Start for Developers](https://www.youtube.com/watch?v=WZY_in9oAjA)

1. [How to build actions.json- Actions package](https://developers.google.com/assistant/conversational/df-asdk/reference/action-package/rest/Shared.Types/ActionPackage)

1. [Building with Actions SDK](https://developers.google.com/assistant/conversational/df-asdk/actions-sdk/define-actions)

1. [gactions steps](https://developers.google.com/assistant/conversational/df-asdk/actions-sdk/gactions-cli)

1. [Gactions and Firebase Setup] (https://developers.google.com/assistant/conversational/quickstart)

1. [gactions CLI](https://developers.google.com/assistant/actionssdk/gactions)

NOTE: Object_Recogniser and cameraActions containers will be required in all methods by default. Current process is for building assistant container.

## Steps

1. Create a new project in actions console.

1. Complete [Device registration](https://developers.google.com/assistant/sdk/guides/service/python)

1. You can create the intents manually in Dialog flow, otherwise in the project directory on the system, you can code the actions by creating actions.json, which can then be pushed to the actions console by gactins command.

1. [What's the difference between ActionsSdkApp and DialogflowApp for Google Assistant?](https://stackoverflow.com/questions/47876438/whats-the-difference-between-actionssdkapp-and-dialogflowapp-for-google-assista/47876688)

1. If you are creating intents using actions.json then create it using the above mentioned resources. (Or can edit a sample actions.json file)

1. Make sure to start writting actions.json from manifest. Check the example given [here](https://developers.google.com/assistant/sdk/guides/service/python/extend/custom-actions)

1. Next, in the project directory terminal, run.
   ```bash
   firebase init functions
   ```
1. Choose the existing project

1. Choose either the typscript or javascript option.

1. Go to the functions directory
    ```bash
    cd functions
    ```
1. Install the actions-on-google sdk. This will help us format the responses that go back to dialog flow.(i.e to build how the assistant reponds to user commands- basically its the webhook/fulfillment)
   ```bash
   npm i actions-on-google --save
   ```
1. Next, we write all the code in the index.ts(typscript option) or index.js(javascript option) file which is already there in functions directory. The code will describe how the assistant reponds to user commands- basically its webhook/fulfillment)
   
1. Based on whether it is typscript or javascript and whether you want to manually create intents in dialogflow or push them via actions console, the import statements will change. Make sure to do the right imports.
   
   * index.ts(typscript) and manually create intents in dialogflow (hence no need to create in actions.json)
   ```bash
   import * as functions from 'firebase-functions';
   import { dialogflow, SimpleResponse, BasicCard, Button/**, other imports**/ } from 'actions-on-google';
   
   const app = dialogflow({ debug: true});
   ```
   
   * index.ts and create intents via actions.json
   ```bash
   import * as functions from 'firebase-functions';
   import { actionssdk/**, other imports**/ } from 'actions-on-google';
   
   const app = actionssdk({ debug: true});
   ```
   
   * index.js and manually create intents in dialogflow (hence no need to create in actions.json)
   ```bash
   const { dialogflow } = require('actions-on-google');
   const functions = require('firebase-functions');
   
   const app = dialogflow({debug: true});
   ```

   * index.js and and create intents via actions.json
   ```bash
   const { actionssdk } = require('actions-on-google');
   const functions = require('firebase-functions');
   
   const app = actionssdk({ debug: true});
   ```
   * In place of actionssdk(actions.json) you can also use. (actions.json file is required here)
   ```bash
   const { conversation } = require('@assistant/conversation');
   const functions = require('firebase-functions');

   const app = conversation({debug: true});
   ```
1. Complete the index.ts/index.js and actions.json

1. Open up the terminal in project directory, run
   ```bash
   firebase deploy --only functions
   ```
1. If the project is successfully deployed, go to the firebase console by click on the link on the terminal.

1. Copy the fulfillment url.

      * If you are manually creating intents in dialogflow, then switch to the fulfillment tab of dialogflow and paste it as the webhook usrl for fulfillment.

      * Otherwise if you are pushing intents via actions.json, then paste it in the (conversations-> url) field of actions.json

      ```
      "conversations": {
          "face-recogniser": {
              "name": "face-recogniser",
              "url": " https://e62f57e859a3.ngrok.io",
              "fulfillmentApiVersion": 2
          }
      },
      ```
1. Now, if you are pushing intents via actions.json, then save your Action package to Google by using the gactions CLI. Replace project_id with your Actions Console project ID

     ```bash
     ./gactions update --action_package actions.json --project <project_id>
     ```
1. To use the update argument, you must use your Action's Project ID. You can get your project ID by clicking the settings gear in your Actions on Google project, followed by Project Settings.

1. The first time you run this command you will be given a URL and be asked to sign in. Copy the URL and paste it into a browser (this can be done on any system). The page will ask you to sign in to your Google account. Sign into the Google account that created the project in a previous step.

   ```bash
   Enter the authorization code:
   ```
   If authorization was successful, you will see a response similar to the following:
   ```bash
   Your app for the Assistant for project my-devices-project was successfully
   updated with your actions.
   ```
1. If you are pushing intents via actions.json, open the Actions console, you will notice before there were no actions but now there are, which has happened due to pushing of actions.json by "gactions update..." command
   
1. Go to actions console -> TEST

1. Give command "talk to my test app" or "talk to invocation name"

1. Give other commands according to training phrases in each intent to test the respective intents.


## NOTE: 
1. actions.json can be pushed to actions console only work if gactions has update command option.
1. For this, if we change node version to '8' in package.json, we can run without cloud permission.
1. Requires cloud permission, if node version is '10'
