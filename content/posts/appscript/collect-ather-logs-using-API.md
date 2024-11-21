---
title: 'Collect Ather Ride Logs using API'
date: 2024-03-17
updated: 2024-10-05
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

It has been 6+ months since the OCR based telegram bot was setup and opened to users. There were 60+ users using the bot and few of them setup their own bot at a time and the count started reducing to single digit. I expected that as long as there is any interactions required from users, it will not work out.

I started setting up the API based bot 3+ months ago and showed how to setup their own in the telegram group. Some are using it and found it is useful.

I want to write a proper documentation how to set it up so that it would be easy to implement. This bot will not be available centrally as it contains lot of personal information and holds lot of data which can't be handled the free usage provided by Google and Telegram as highlighted in the previous post. So, if you plan to use it, you need to get your hands dirty.

The OCR based bot is continue to exist, there's no change on that.

<!--more-->
<!--TOC-->

## Purpose

This is just the enhanced version of the previous OCR based bot, with less uers interaction with more data such as ride mode, braking, coasting distance, etc.,

The Ather mobile app stores only the last 20 rides. If the a person uses OCR based bot then (s)he can get the last 20 rides only. The automated bot extracts all the historical data from the vehicle purchase time.

One important thing to note, Ather might block this api access in future. So it may or may not work. They already blocked api access for any triggers done from Google App script as of 5th Oct 2024. I'm updating a workaround how to trigger this api from other sources.

## Requirements

1. You should be the owner of Ather Vehicle and you should have the option (password or OTP) to login to Ather mobile app.
2. An Android phone with USB debugging enabled.
3. A PC with Android Debug Bridge (ADB) installed (Google it how to install adb tools on pc)
4. Ather connect subscription.
5. A Google account to store the ride details.
6. A telegram account to interact with the data and getting alerts.
7. A home server or a Github account to trigger the api, as Ather blocks the api access from Google App script.
8. Patience bro!

{{< alert >}}
To access the ride log records, you need to get access to the Ather's FireBase database using your Ather mobile app. Don't share your login credentials to any anyone unless you completely trust them.
{{< /alert >}}

## How It Works

1. User takes a ride, the relevant data would be shared to Ather for their analysis.
2. Ather stores the data in their FireBase database.
3. A script retrieves the data using API endpoints on a periodical interval and stores it in Google sheet.
4. Telegram bot send an alert to the owner when there is an update on new ride.
5. In addition, an user can interact with bot for charts and analysis.

## Setup

### Get API Key from Ather app

1. **Windows User**: Make sure you have installed ADB drivers on your PC, then download the platform tools zip file from [Google's android developers website](https://developer.android.com/tools/releases/platform-tools). You may find lot of resources in the internet and youtube videos to complete it.
2. **Linux User**: Use `sudo apt install adb fastboot` on debian based PC; `sudo pacman -S android-tools` on arch based PC.
3. Go to your Android Phone > Settings > About Phone > Build Number > Tap 7 times till you get a message that shows that **You are now a developer**. This may be available in different places, depends on the phone manufacturer.
4. **Windows User**: Open command prompt or terminal on your PC and go to the platform tools folder.
5. Connect your phone to your PC using an USB cable, then run `adb devices` command in the terminal window.
6. You should get a message on your mobile phone to accept the authorization from your PC. Accept it.
7. Run `adb devices` command once more. It should display your mobile phone's ID then a word `device`. You shouldn't be seeing unauthorized next to the phone ID, which represents that the phone is not authorized it yet.
8. Run `adb logcat | grep Bearer` command if you are using linux; `adb logcat -d | findstr Bearer` command if you are using windows.
9. Open Ather mobile app and go to the **Support** section on the bottom (should be on the 3rd item), then go to `My ticketes` Check for the below output.
10. `<timestamp> <pid>  <pid>` I System.out: Cosmo Log:: -> Authorization: Bearer <400+ character random text>

{{< alert >}}
The 400+ random character is your API **TOKEN**. Keep it safe for future use; do not share it with anyone.
{{< /alert >}}

### Get your Vehicle Identification Number (VIN)

This should be available in your RC book or in the boot or in the vehicle dashboard.

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

### Google Slide

Create a new Google slide file in your Google Drive.
![google-slide-url](/images/ather-log/google-slide-url.png)

{{< alert >}}
Get the ID of the slide and keep it for later use. Lets call it as **SLIDE_ID**.
{{< /alert >}}

### Google Sheets

Make a copy of this [Google Sheet (refer readme)](https://github.com/ev-log-bot/ev-log-bot/tree/auto).
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
![google-sheet-options-tab](/images/ather-log/google-sheet-options-tab_2.png)

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

Choose your gmail ID, if you don't want to use your primary gmail you can create/use a new one.
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

Now go to `7_other_func.gs` on the left panel, select UpdateScriptProperties function from drop down then click Run. This will update the script properties that you fill in 'options' sheet.
![script-update-script-properties](/images/ather-log/script-update-script-properties_2.png)

The above step can be manually done by clicking the gear icon on the left pane, which will open up the project settings.
![script-project-settings](/images/ather-log/script-project-settings.png)

Scroll to the bottom where you can see the Script Properties.

Open the telegram app and open the bot that you have created. If you don't know the bot, the link would be available in BotFather. Hit 'Start' button.

### Setup Triggers

As mentioned earlier Ather blocks accessing the api from the Google Appscript. So triggering the api in order to get the data should be done outside the Google Sheets.

Follow any of the below steps to populate the data in the google sheets.

#### Option 1: Github Actions

1. Create a [github](https://github.com/) account, if you don't have one.
2. Go to [ev-log-bot-addon](https://github.com/ev-log-bot/ev-log-bot-addon/fork) repo and fork it into your github account.
3. Go to Settings (should be available on the top) of the repo, then Secrets and Variables (should be available on the left) and click Actions.
4. Under Secrets > Repository Secrets, click New Repository Secret button.
5. You need to create 3 secrets (`API_TOKEN`, `SCOOTER_ID`, `WEBHOOK_URL`) with the relevant information.
6. The name of the secrets are case sensitive, don't make any mistakes.
7. Go to `Actions` (should be on top in the same level of Settings) and click `I understand my workflows, go ahead and enable them' button.
   ![Github Secrets](/images/ather-log/github-secrets.png)
   ![Github Actions](/images/ather-log/github-actions.png)
   ![Github enable workflow](/images/ather-log/github-enable-workflow.png)

The code would retrieve the ride details for every 23 minutes. This is calculated considering the 2000 free minutes offered by Github. Decreasing the time would result in hitting the limit set by Github.

{{< alert >}}
The above option would work till Ather blocks the IP address of Github.
{{< /alert >}}

#### Option 2: Using a home server or a pc

1. Clone [ev-log-bot-addon](https://github.com/ev-log-bot/ev-log-bot-addon/) repo to your home server.
2. Make a copy of `.env_example` as `.env` and update the 3 variables that are relevant to your vehicle and your Google sheets.
3. Suggest you to create a virtual env using venv, the code would work fine even without a virtual env.
4. Install the requirements by running the command `pip install -r requiremnets.txt`.
5. Running the `python main.py` would pull the ride details from ather server and updates into Google Sheets.
6. You may schedule a cron job to pull the data on periodical interval.

### Extracting Historical Data if you are a new user

If you analyse the python script on the ev-log-bot-addon is setup to extract the last 20 rides only. Refer this line:

```python
ride_data = get_ride_details(scooter_id, api_token, 20, "desc")
```

You may need to change the above code to extract all historical rides like below:

```python
ride_data = get_ride_details(scooter_id, api_token, 2000, "asc")
```

{{< alert >}}
You need to modifiy the `main.py` file on the Github repo and commit the change. Wait for the next interval to see the changes on the Google Sheets. Once all your historical data is updated, you **must** change the code back to original. **Your rides will not get updated, if you don't update the code back to original.**
{{< /alert >}}

Ather might revoke the api keys on periodical basis. You need to update the api keys on the Environment Variables or Github Actions Secrets when there is a change.

## Usage

Start the bot or send `/start` command to the bot, it would repond with the shortcut keys to interact with the bot.

Following are the shortcut keys (not case sensitive) at this time the blog post is published.

`D` - Daily Charts  
`W` - Weekly Charts  
`M` - Monthly Charts
`DS` - Daily Summary  
`WS` - Weekly Summary  
`MS` - Monthly Summary  
`set A/B/C` - Set Trip A/B/C  
`get A/B/C` - Get Trip A/B/C info  
`AT <token>` - will replace the Ather token  
`SOC value` - set an alert when the SOC drops below the value  
`DASH` - displays the option to get the dashboard

For questions, contact me on this [Telegram channel](https://t.me/ather_india). I'll respond when available, but immediate support isn't guaranteed.

The code behind this process is open and available in {{< icon "github" >}} [github repository](https://github.com/ev-log-bot/ev-log-bot/) under 'auto' branch. Adding a ⭐ would be much appreciated.
