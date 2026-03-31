# n8n
## Problem statment
Many users communicate scheduling information through messaging platforms but fail to record those details in their calendars. This manual process is time-consuming and error-prone.

This project automates the process by allowing users to send text messages to a Telegram bot. 
The system extracts relevant scheduling information and creates a corresponding Google Calendar event, optionally including invited participants and a Google Meet conference link.



## Solution overview
this project is build using n8n for the automation integerating :
+ **Telegram bot** : sends comming message to the n8n workflow
+ **n8n workflow** : process incoming data
+  **Google Calendar api** : for event and meeting creation
+  **Google gmail api** : for sending emails

<img  height="700" alt="image" src="https://github.com/user-attachments/assets/67e8488d-af4a-4761-9e1f-d0fc25bcb686" />

## Implementation 

 ### 1- Telegram Trigger

 <img width="469" height="448" alt="image" src="https://github.com/user-attachments/assets/2abd50a5-a205-436a-9371-2920dd4539f4" />
 

 we used @BotFather at telegram to create the bot
 
 <img width="641" height="356" alt="image" src="https://github.com/user-attachments/assets/048426fa-a913-4042-a646-bc734ff51e8b" />
 
 >at the end of the process you will be givin a key for the bot to be used in telegram trigger node Access Token

 <img width="1196" height="552" alt="image" src="https://github.com/user-attachments/assets/0a3edbbd-e3f3-4df3-b247-d0411976a0e0" />

 ___

 ### 2- if statment ( user check )
 <br>

 <img width="711" height="438" alt="image" src="https://github.com/user-attachments/assets/f81279f2-9a46-45df-be80-012db9cf88aa" />

 >the node will check if the incomng message from a specific user. if not it will return "sorry, you dont have access to this bot"

 * (if node)

 <img width="1256" height="420" alt="image" src="https://github.com/user-attachments/assets/bdc9559f-2c17-426b-8a0b-8678c9037ad3" /> <br> <br> <br>
 
 * (message node) <br> 
 
 <img width="1134" height="651" alt="image" src="https://github.com/user-attachments/assets/16a0261c-f529-4d41-9202-a7680d09406c" /> <br>

___

### 3- LLM parser

<img width="733" height="613" alt="image" src="https://github.com/user-attachments/assets/e4d3b1a3-03e0-4c96-838f-ad4558c7c092" />

 before sending the text to the LLM parser , it will check what is the type of the incoming message. Since it's only accept Text format

 if it not a text it will alert the user 

 the LLM parser will take the given text and determen the property so the next nodes can operates
 the output will be in **JSON**

 <img width="1870" height="923" alt="image" src="https://github.com/user-attachments/assets/88ba27c2-3ff0-40a5-8249-e87d709ab32b" />

 #### **the LLM agent properties are** :

 * **Chat Model**: gemini 2.5 flash

 * **AI Agent Promt** : 
```
you are a task extractor. You will be given a text to extract the needed information from it 
do not add from yourself

you will decide if the give text is about an online meeting where it should be with other participant (a team) where it is going to be hosted on Google Meet, or a personal event or a personal meeting that doesn't require an online meeting
for the meeting is should a group meeting if not then it is personal

The output should be in JSON format.

{
  "summary": "Strategy Meeting",
  "location": "Eiffel Tower, Paris, France",
  "description": "A meeting to discuss the new project strategy.",
  "start": {
    "dateTime": "2026-03-10T08:00:00.000+03:00",
    "timeZone": "Asia/Riyadh"
  },
  "end": {
    "dateTime": "2026-03-10T09:00:00.000+03:00",
    "timeZone": "Asia/Riyadh"
  },
  "email" : "faisal2341234@gmail.com"
  "Group_meeting" : true,
  "online_meeting" : true
}

- the current date is {{ $now }}
-The time should be in ISO and have the same format
-The timezone is: Asia/Riyadh
-Do not add any information from yourself
-only produce JSON, 
-do not add any new parameters

AND DO NOT RETURN TEXT
If the location is not provided, make it "null"
if the email is not provided make it "null"
you can make the description better grammatically, but don't add any new information

If no valid event information is found, return:

{
  "summary": "null",
  "location": "null",
  "description": "null",
  "start": {
    "dateTime": "null",
    "timeZone": "Asia/Riyadh"
  },
  "end": {
    "dateTime": "null",
    "timeZone": "Asia/Riyadh"
  },
  "email": "null"
}


THE MESSAGE:
{{ $json.message.text }}
```
* **OutPut Parser** :

>the Scheme type is a JSON example

<img width="417" height="666" alt="image" src="https://github.com/user-attachments/assets/3c41ae68-fade-497c-a1d9-cba29abca6d0" />


```
{
  "summary": "Strategy Meeting",
  "location": "Eiffel Tower, Paris, France",
  "description": "A meeting to discuss the new project strategy.",
  "start": {
    "dateTime": "2026-03-10T08:00:00.000+03:00",
    "timeZone": "Asia/Riyadh"
  },
  "end": {
    "dateTime": "2026-03-10T09:00:00.000+03:00",
    "timeZone": "Asia/Riyadh"
  },
  "email" : "faisal2341234@gmail.com",
  "Group_meeting" : true,
  "online_meeting" : true
}

```
___

after that the output will branch to two sections

<img width="598" height="481" alt="image" src="https://github.com/user-attachments/assets/1e98d752-1b60-4fa1-b4e1-dfe020a8c888" />

but before that an if statment will check any values are not present 

<img width="221" height="230" alt="image" src="https://github.com/user-attachments/assets/4ee1b6ec-fdfe-4858-81a7-8e437c1d480a" />


___


##  * Google Calendar 

<img width="977" height="484" alt="image" src="https://github.com/user-attachments/assets/fbf78bde-5991-408e-bec4-d18beec30991" />

the if statment will check if the value for the parameter meeging in JSON is **true** or not

<img width="646" height="127" alt="image" src="https://github.com/user-attachments/assets/41f514ec-2062-4341-be5e-1f3c26800c80" />

if it a an online meeting 
this node will execude 

<img width="1867" height="882" alt="image" src="https://github.com/user-attachments/assets/806e9026-9c25-410b-a609-3184b2e2f441" /> <img width="401" height="337" alt="image" src="https://github.com/user-attachments/assets/38c8d0e5-1552-4e07-8526-88278a076689" />

if it not the other one will execude

<img width="1865" height="889" alt="image" src="https://github.com/user-attachments/assets/c3a49df2-c82e-4e50-a9c2-e7572951611f" />


## *Google Gmail

<img width="778" height="359" alt="image" src="https://github.com/user-attachments/assets/cb09bb82-d63f-4049-8267-27f487b3a8c2" />

>the if statment will check if the  **Email** property is "null" or not

<img width="647" height="160" alt="image" src="https://github.com/user-attachments/assets/3c7748f4-fe3a-4f07-a2a9-fc584b2676a2" />

*if it not "null"
<br><br><br>

it will send an email to the given email from the text 
that include 

* summery
* description

  
<img width="422" height="615" alt="image" src="https://github.com/user-attachments/assets/7802bec2-8a86-4181-a523-f22b58020603" />

# How to Run

ther are two to run n8n 
* on the cloud through n8n
* locally

i will show the steps to for setup n8n locally using Docker

### requirements
  * installing Docker
  * downloading n8n image through Docker
  * optionally : if you are planning to use webhooks and triggers you **Must** a **_Domain_** and a **_Tunnel_** for the the domain and download the image from docker 

 <br><br><br>

 #### Docker Compose 
 i used **Docker compose** to create the containers 
 
 the name of the file should be `docker-compose.yml` 

 ```
version: "3.8"

services:
    n8n:
        image: n8nio/n8n:latest
        container_name: n8n
        restart: unless-stopped
        ports:
            - "5678:5678"
        environment:
            - N8N_HOST= << THE URL FOR THE DOMAIN >> 
            - N8N_PROTOCOL=https
            - WEBHOOK_URL=https:  << THE URL FOR THE DOMAIN >> 
            - TZ=Asia/Riyadh
            - GENERIC_TIMEZONE=Asia/Riyadh
            - N8N_ENABLE_COMMUNITY_NODES=true
        volumes:
            - n8n-data:/home/node/.n8n

    << cloudflare is optional unless you want to access your n8n intance throught the internet or have webhooks >> 
        
    cloudflared:
        image: cloudflare/cloudflared:latest
        container_name: cloudflared
        restart: unless-stopped
        command: tunnel --no-autoupdate run --token /<< YOUR TUNNEL TOKEN >>
        depends_on:
            - n8n

volumes:
    n8n-data:

 ```

 >a domain is necessery to enable communication with webhooks

___
many nodes will ask for credintials. to use them you must provide the needed credintials 

<br>
<hr>
<br> 
<img width="1876" height="954" alt="image" src="https://github.com/user-attachments/assets/9b1000a8-9e39-4e88-b1db-42377fdc344f" />

 > we have added a new google calender node and an if statment to decided if the it an online meeting or not

 


https://github.com/user-attachments/assets/76f1806c-220d-4093-92d6-e10d59b14317


  
