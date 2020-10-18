## Method 1: 
Previously, in the flask assistant, we wrote the assistant(webhook) using flask_assistant package. However, for the assistant to work, we had to manually create intents/entities in the Dialogflow interface, to which the assistant reply. The assistant.py(webhook) contains only the implementation of how the assistant reply/reacts to a intent activated by a command given by the user.

This method can be built upon the Flask assistant, where instead of manually creating intents and entities and other features in Dialogflow. actions.json file can be written, where we describe the creation of intents/entities/other features in dialogflow. If we do "gactions update..." on this actions.json file, all the intents and other features will be updated automatically. 

Except for creating a new project and device registration in actions console, there is no other requirement of interacting with actions console. You dont have to even give the invocation name manually in actions console. Everything will be done by actions.json. 

Basically, actions.json is like a backend version of actions console UI.

Requires gactions to have the command options: 

* gactions update...

* gactions test...

NOTE: This is the official method given by documentation after device registration.

## Resources
1. [Register custom Device actions](https://developers.google.com/assistant/sdk/guides/service/python/extend/custom-actions)

1. [How to build actions.json- Actions package](https://developers.google.com/assistant/conversational/df-asdk/reference/action-package/rest/Shared.Types/ActionPackage)

1. [Building with Actions SDK](https://developers.google.com/assistant/conversational/df-asdk/actions-sdk/define-actions)

1. [gactions steps](https://developers.google.com/assistant/conversational/df-asdk/actions-sdk/gactions-cli)

1. [Building Google Assistant Apps with Python Demo| Paul Bailey @ PyBay2018](https://www.youtube.com/watch?v=5eRxMyf_2Rc)

1. [Sample App](https://github.com/pizzapanther/google-actions-python-example)

## Server Setup

This Assistant will use ngrok to quickly provide a public URL for the flask-assistant webhook. This is required for Dialogflow to communicate with the assistant app.

#### **ngrok**

Install ngrok from [install guide](https://dashboard.ngrok.com/get-started/setup) for Mac OS. 


Then, enter the following commands
```bash
unzip /path/to/ngrok.zip
```

```bash
./ngrok authtoken 1iK0ofMlVmYeN0zIp1kTa0WouPD_5njYQVBzD9NxWMDrRRet4
```

Once ngrok is installed, in the project directory's terminal enter

```bash
./ngrok http 5000
```

The server will start. A status message similiar to the one below will be shown. Keep this terminal open.

```bash
ngrok by @inconshreveable                                                                

Session Status                online
Version                       2.1.18
Region                        United States (us)
Web Interface                 http://127.0.0.1:4040
Forwarding                    http://1ba714e7.ngrok.io -> localhost:5000
Forwarding                    https://1ba714e7.ngrok.io -> localhost:5000
```
Note the Forwarding https URL.

* https://1ba714e7.ngrok.io in the above example.

* This is the URL that will be used as the Webhook URL in the actions.json(which will build dialog flow) as described below.

* URL needs to be changed in actions.json every time the server is started. 

## Steps

1. Create a new project in actions console.

1. Complete [Device registration](https://developers.google.com/assistant/sdk/guides/service/python)

1. You can create the intents manually in Dialog flow, otherwise create a new empty project directory, in the project directory, you can create actions.json which can be pushed to the actions console by gactins command.

1. If you are creating intents using actions.json then create it using the above mentioned resources. (Or can edit a sample actions.json file)

1. Make sure to start writting actions.json from manifest. Check the example given [here](https://developers.google.com/assistant/sdk/guides/service/python/extend/custom-actions)

1. Next, in the project directory, create webhook.py(any_name.py) which will contain how the assistant responds to the different activated intent based on user command. (webhook.py or the fulfillment will be same for all methods if the intent names are constant)

1. Start the ngrok server, if not already started.

    ```bash
    ./ngrok http 5000
    ```
1. Copy the forwarding https url, and paste it in the (conversations-> url) field of actions.json

    ```
    "conversations": {
        "face-recogniser": {
            "name": "face-recogniser",
            "url": " https://e62f57e859a3.ngrok.io",
            "fulfillmentApiVersion": 2
        }
    },
    ```
1. Save your Action package to Google by using the gactions CLI. Replace project_id with your Actions Console project ID

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
1. Open Actions console, you will notice before there were no actions but now there are, which has happened due to pushing of actions.json by "gactions update..." command(due to above command)

1. Return to the project directory, in the terminal, run your webhook.py(application)
   ```bash
   python3 webhook.py
   ```
   
1. Go to actions console -> TEST

1. Give command "talk to my test app" or "talk to <invocation name>"

1. Give other commands according to training phrases in each intent to test the respective intents.


## NOTE: 
1. This method will only work if gactions has update command option.
1. Requires cloud permission, if node version is '10'
