---
title: 'Collect Ather Ride Logs using API'
date: 2024-03-17
updated: 2024-03-17
categories:
  - projects
tags:
  - ev
  - ather
  - google-sheets
  - telegram-bot
  - appscript
  - api
keywords:
  - ev
  - ather
  - google-sheets
  - telegram-bot
  - appscript
comments: true
thumbnailimage: 'images/ather-log/ather-logo.webp'
TOC: true
---

It has been 6+ months since the OCR based telegram bot was setup and opened to users. There were 60+ users using the bot and few setup their own bot at a time and the count started reducing to single digit. I expected that as long as there in any interactions from users, it will not work out.

I started setting up the API based bot 3+ months go and showed how to setup their own in the telegram group. Some are using it and found it is useful.

I want to write a proper document how to set it up so that it would be easy to implement. This bot will not be available centrally as it contains lot of personal information and holds lot of data which doesn't handle the free usage provided by Google and Telegram as highlighted in the previous post. So, if you plan to use it, you need to get your hands dirty.

The OCR based bot is continue to exist, there's no change on that.

<!--more-->
<!--TOC-->

## Purpose

This is just the enhanced version of the previous OCR based bot, with less uers interaction with more data such as ride mode, braking, coasting distance, etc.,

The Ather mobile app stores only the last 20 rides. If the a person uses OCR based bot then (s)he can get the last 20 rides only. The automated bot extracts all the historical data from the vehicle purchase time.

One important thing to note, Ather might block this api access in future. So it may or may not work. They already blocked api access for any manual triggers that are done from Google App script.

## Requirements

1. You should be the owner of Ather Vehicle and you should have the option (password or OTP) to login to Ather mobile app.
2. An Android phone with USB debugging enabled.
3. A PC with Android Debug Bridge (ADB) installed (Google it how to install adb tools on pc)
4. Ather connect subscription.
5. A google account to store the ride details.
6. A telegram account to interact with the data and getting alerts.
7. Patience bro!

{{< alert >}}
To access the ride log records, you need to get access to the Ather's FireBase database using your Ather mobile app. Don't share your login credentials to any anyone unless you completely trust them.
{{< /alert >}}

## How It Works

1. User takes a ride, the relevant data would be shared to Ather for their analysis.
2. Ather stores the data in their FireBase database.
3. A script retrieves the data using API endpoints on a periodical interval and stores it in Google sheet.
4. Telegram bot send an alert to the owner when there is an update / new ride.
5. In addition, user can interact with bot for charts and analysis.

## Setup

### Get API Key from Ather app

1. **Windows User**: Make sure you have installed ADB drivers on your PC, then download the platform tools zip file from [Google's android developers website](https://developer.android.com/tools/releases/platform-tools). You may find lot of resources in the internet and youtube videos to complete it.
2. **Linux User**: Use `sudo apt install adb fastboot` on debian based PC; `sudo pacman -S android-tools` on arch based PC.
3. Go to your Android Phone > Settings > About Phone > Build Number > Tap 7 times till you get a message that shows that **You are now a developer**. This may be available in different places, depends on the phone manufacturer.
4. **Windows User**: Open command prompt or terminal on your PC and go to the platform tools folder.
5. Connect your phone to your PC using an USB cable, then run `adb devices` command in the terminal window.
6. You should get a message on your mobile phone to accept the authorization from your PC. Accept it.
7. Run `adb devices` command once more. It should display your mobile phone's ID then a word `device`. You shouldn't be seeing unauthorized next to the phone ID, which represents that the phone is not authorized it yet.
8. Run `adb logcat | grep user_cb_token` command if you are using linux; `adb logcat | findstr user_cb_token` command if you are using windows.
9. Open Ather mobile app and go to the **Charger** section on the bottom (should be on the 2nd item). Check for the below output.
10. `<timestamp> <pid>  <pid>` D user_cb_token: Bearer <400+ character random text>

{{< alert >}}
The 400+ random character is your API **TOKEN**. Keep it safe for future use; do not share it with anyone.
{{< /alert >}}

### Get your Vehicle Identification Number (VIN)

This should be available in your RC book or in the boot.

{{< alert >}}
Refer this as **VIN**.
{{< /alert >}}

### Setting up Telegram Bot

1. Search for BotFather on Telegram or use this link: [https://t.me/BotFather](https://t.me/BotFather).
2. Start a chat and choose `/newbot` from the menu.
3. Provide basic information such as Bot's name and username (naming rules apply).

![Botfather](/images/ather-log/botfather.png)

{{< alert >}}
Refer this API Token as **BOT TOKEN**.
{{< /alert >}}

### Get your Telegram Numeric ID

Go to https://t.me/userinfobot on your telegram client and say hi. You will get a numeric value.
{{< alert >}}
Refer this as **ADMIN** ID.
{{< /alert >}}

### Get API Key from Bing Maps

Go to https://www.bingmapsportal.com/ login with your Microsoft Account. Follow the steps to get an API key.
This will help you to get the location name of the start and end place.

{{< alert >}}
Refer this as **MAPS_KEY**.
{{< /alert >}}

### Google Slide

Create a new Google slide file in your Google Drive.
![google-slide-url](/images/ather-log/google-slide-url.png)

{{< alert >}}
Get the ID of the slide and keep it for later use. Lets call it as **SLIDE_ID**.
{{< /alert >}}

### Google Sheets

Make a copy of this Google Sheet: [Google Sheets Template](https://docs.google.com/spreadsheets/d/1PAL8Qf-AsHOC9s4Ddh_oO4DSCrHIhoMkFSoqVS_kiuM).
![google-sheet-make-a-copy](/images/ather-log/google-sheet-make-a-copy.png)

It will ask you to name the file and also highlight that the scripts also will be saved.
![google-sheet-make-copy-name](/images/ather-log/google-sheet-make-copy-name-new.png)

You can rename the sheet the way you want and click 'Make a copy'. Once the file is saved, you will be able to make edits to the sheet. **Make sure you check copy comments option.**

### Know your Vehicle to know the SOC Capacity

Go to the **options** sheet and find the battery capacity from range A11 to B17

{{< alert >}}
Refer this as **SOC_CAPACITY**.
{{< /alert >}}

### Fill the details in options Sheet

Go to `options` sheet and fill the details on yellow highlighted cell (from cell B2 to B8) that you got from above steps.
Keep the `WEBHOOK_URL` as blank.
![google-sheet-options-tab](/images/ather-log/google-sheet-options-tab.png)

### Setting up Script

Access Google Apps Script via the "Extensions" menu in Google Sheets.
![google-app-script-start](/images/ather-log/google-app-script-start.png)
Rename the project to match your sheet's name (optional).
Deploy the script, authorizing necessary permissions.
![script-deploy-button](/images/ather-log/script-deploy-button.png)

You will get a pop-up like below. Make sure you select the highlighted poritions. If you make mistakes here, your bot will not work.

![script-deploy-1st](/images/ather-log/script-deploy-1st.png)

Then you need to follow the steps as highlighted in the screenshots.
These steps are required to access your google drive, google sheets, google docs and slides, to process further.

![script-authorize-access](/images/ather-log/script-authorize-access.png)

Choose your gmail ID, if you don't want to use your primary gmail you can create a new one.
![script-authorize-gmail](/images/ather-log/script-authorize-gmail.png)

You will get a warning that the app is unverified. Click advanced
![script-authorized-advanced](/images/ather-log/script-authorized-advanced.png)

Click go to 'your google sheet file name'
![script-authorize-unsafe](/images/ather-log/script-authorize-unsafe.png)

Click Allow
![script-authorize-allow](/images/ather-log/script-authorize-allow.png)

You will get a deployment URL. Keep it safe and do not share it with anyone.
![script-deploy-copy-url](/images/ather-log/script-deploy-copy-url.png)

{{< alert >}}
Refer this URL as **WEBHOOK_URL** and update the same in the Options sheet.
{{< /alert >}}

### Multiple deployments (Not required for first time)

Do not use `New Deployment` multiple times, if you do the WEBHOOK_URL will change.

If you wish to deployment again go to `Deploy > Manage Deployments > Click Pencil Icon > Version to New Version > Deploy`. This will retain the WEBHOOK_URL as same.

![script-deploy-manage-deployments](/images/ather-log/script-deploy-manage-deployments.png)
![script-deploy-new-version](/images/ather-log/script-deploy-new-version.png)

{{< alert >}}
**IMPORTANT** This step has to be done whenever we change something on the code or script properties.
{{< /alert >}}

### Update Script Properties

Now go to `Misc.gs` on the left panel, select UpdateScriptProperties function from drop down then click Run. This will update the script properties that you fill in 'options' sheet.
![script-update-script-properties](/images/ather-log/script-update-script-properties.png)

The above step can be manually done by clicking the gear icon on the left pane, which will open up the project settings.
![script-project-settings](/images/ather-log/script-project-settings.png)

Scroll to the bottom where you can see the Script Properties.

### Setup Webhook

Go to Script editor from the left pane.
![script-editor](/images/ather-log/script-editor.png)

Then go to `bot.gs` file.
You will see many functions like `getMe()`, `setWebhook()`, etc.,

from the top select `getMe` then click Run. You will get a success message in the execution log on the bottom.
![script-run-get-me](/images/ather-log/script-run-get-me.png)

Now do the same for `setWebhook`.
![script-run-set-webhook](/images/ather-log/script-run-set-webhook.png)

{{< alert >}}
if you are getting a result as 'ok: true' then, you have followed the steps without any mistakes.
{{< /alert >}}

Run `deleteWebook` function if you are using the same bot with a different google sheet. Then do the `setWebook`. Once the webhook setup is complete the telegram bot that you have created above will start working.

Open the telegram app and open the bot that you have created. If you don't know the bot, the link would be available in BotFather. Hit 'Start' button.

### Setup Triggers for extracting Historical Data

Click on the left panel and click Triggers. Then click Add Trigger button.
![script-add-trigger](/images/ather-log/script-add-trigger.png)

Create a trigger to trigger the **firstRun** function as mentioned below. Enter current date and current time + 2 minutes in the 'Enter Date/Time' field.
![script-trigger-first-run](/images/ather-log/script-trigger-first-run.png)

{{< alert >}}
Google app script would run for maximum 90 seconds. The function would extract approximately 1400 lines of data from the server in that time. You need to schedule it again till you get the latest ride (refer data sheet in Google sheet).
{{< /alert >}}

If you get any errors in this step, your ather TOKEN or MAPS_KEY might be wrong.

### Setup Triggers for Ride Alert

Once you get the historical data, click 'Add Trigger' again and setup trigger for 'triggerApi' function. Now the 'Select type of time based trigger' field should be 'Minutes Timer' and minute interval should be 10 minutes.
![script-trigger-api](/images/ather-log/script-trigger-api.png)

{{< alert >}}
Running these scripts directly from the App Script window would fail, as ather has already blocked those IP address. It works only by scheduling, until ather blocks them.
{{< /alert >}}

### Usage

Start the bot or send `/start` command to the bot, it would repond with the shortcut keys to interact with the bot.

Following are the shortcut keys (not case sensitive) at this time the blog post is published.

`D` - Daily Charts  
`W` - Weekly Charts  
`M` - Monthly Charts  
`T` - Trigger API & enable Triggers  
`G` - Get API Status  
`O` - Toggele Triggers ON or OFF  
`DS` - Daily Summary  
`WS` - Weekly Summary  
`MS` - Monthly Summary  
`set A/B/C` - Set Trip A/B/C  
`get A/B/C` - Get Trip A/B/C info  
`AT <token>` - will replace the Ather token'

{{< alert >}}
I strongly suggest that disable the api triggers when you are not using the vehicle by sending command `O`. So that, we don't overuse the api access and its win-win for users and Ather.
{{< /alert >}}

For questions, contact me on this [Telegram channel](https://t.me/ather_india). I'll respond when available, but immediate support isn't guaranteed.

The code behind this process is open and available in {{< icon "github" >}} [github repository](https://github.com/anbuchelva/ev-log-bot/) under 'auto' branch. Adding a ⭐ would be much appreciated.
