### Method 1-
Previously, in the flask assistant, we wrote the assistant(webhook) using flask_assistant package. However, for the assistant to work, we had to manually create intents/entities in the Dialogflow interface, to which the assistant reply. The assistant.py(webhook) contains only the implementation of how the assistant reply/reacts to a intent activated by a command given by the user.

This method can be built upon the Flask assistant, where instead of manually creating intents and entities and other features in Dialogflow. actions.json file can be written, where we describe the creation of intents/entities/other features in dialogflow. If we do "gactions update..." on this actions.json file, all the intents and other features will be updated automatically. 

Except for creating a new project and device registration in actions console, there is no other requirement of interacting with actions console. You dont have to even give the invocation name manually in actions console. Everything will be done by actions.json. 

Basically, actions.json is like a backend version of actions console UI.

Requires gactions to have the command options: 

gactions update....

gactions test...

NOTE: This is the official method given by documentation after device registration.

## Resources
1. [Register custom Device actions](https://developers.google.com/assistant/sdk/guides/service/python/extend/custom-actions)

1. [How to build actions.json- Actions package](https://developers.google.com/assistant/conversational/df-asdk/reference/action-package/rest/Shared.Types/ActionPackage)

1. [Building with Actions SDK](https://developers.google.com/assistant/conversational/df-asdk/actions-sdk/define-actions)

1. [gactions steps] (https://developers.google.com/assistant/conversational/df-asdk/actions-sdk/gactions-cli)

1. [Building Google Assistant Apps with Python Demo| Paul Bailey @ PyBay2018] (https://www.youtube.com/watch?v=5eRxMyf_2Rc)



## Steps

1. Create a new project in actions console

1. [Device registration](https://developers.google.com/assistant/sdk/guides/service/python)

1. create actions.json using the above mentioned resources. (Or can edit a sample axtions.json file)

1. Make sure to start actions.json from manifest. Check the example given [here](https://developers.google.com/assistant/sdk/guides/service/python/extend/custom-actions)

1. create webhook.py(any_name.py) which will contain how the assistant responds to the different activated intent based on user command. webhook.py will be same for all methods
