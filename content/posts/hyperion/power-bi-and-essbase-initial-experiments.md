---
title: Power BI and Essbase - Initial Experiments
date: 2018-12-23T19:18:37.727Z
updated: 2019-06-27 2:18:37 UTC
categories:
  - Hyperion
tags:
  - power bi
  - essbase
  - connector
keywords:
  - Power BI
  - Essbase
  - Connector
  - Connector
thumbnailimage: "https://res.cloudinary.com/anbuchelva/image/upload/c_scale,h_250,f_auto,q_auto/v1579969704/images/power-bi/powerbi_logo.png"

---
Please go through my previous post that explains [how to connect Essbase with Power BI using the Essbase (beta) connector](/hyperion/power-bi-steps-to-connect-essbase-using-essbase-connector-beta/), if you are getting your hands in the first time.
<!--more-->
I have tried connecting to two different ASO cubes and share how Essbase reacts while creating a new model in Power-BI.



Initially I have tried connecting the `ASOsamp` application after loading some random values as shown in the screenshot.

![Power_BI_Essbase_Connector_Initial_Experiments](https://res.cloudinary.com/anbuchelva/image/upload/f_auto,q_auto/v1546629705/images/power-bi/asosamp-dataload.png)

I have tried selecting each dim from the bottom, it generated the table as displayed in the following screenshot.

![Power_BI_Essbase_Connector_Initial_Experiments](https://res.cloudinary.com/anbuchelva/image/upload/f_auto,q_auto/v1546629705/images/power-bi/Power_BI_Essbase_Initial_Experiments_2.png)

It started freezing when I select all the dims and made the server to crash _(I setup a VM with 8 GB RAM, which is not enough to process the request),_ considering the ASOsamp is a small application compared to the production applications.

![Power_BI_Essbase_Connector_Initial_Experiments](https://res.cloudinary.com/anbuchelva/image/upload/f_auto,q_auto/v1546629705/images/power-bi/Power_BI_Essbase_Initial_Experiments_3.png)

Logs for reference _(recreated for this post)_:  ~~<https://hastebin.com/esazoyiwix.sql>~~.  I prefer not to run it in production and crash the servers.

Then I tried creating very small ASO application and loaded some random data.  This time it worked and I was able to get the values as expected.

![Power_BI_Essbase_Connector_Initial_Experiments](https://res.cloudinary.com/anbuchelva/image/upload/f_auto,q_auto/v1546629705/images/power-bi/Power_BI_Essbase_Initial_Experiments_4.png)
![Power_BI_Essbase_Connector_Initial_Experiments](https://res.cloudinary.com/anbuchelva/image/upload/f_auto,q_auto/v1546629705/images/power-bi/Power_BI_Essbase_Initial_Experiments_5.png)
![Power_BI_Essbase_Connector_Initial_Experiments](https://res.cloudinary.com/anbuchelva/image/upload/f_auto,q_auto/v1546629705/images/power-bi/Power_BI_Essbase_Initial_Experiments_6.png)
![Power_BI_Essbase_Connector_Initial_Experiments](https://res.cloudinary.com/anbuchelva/image/upload/f_auto,q_auto/v1546629705/images/power-bi/Power_BI_Essbase_Initial_Experiments_7.png)
![Power_BI_Essbase_Connector_Initial_Experiments](https://res.cloudinary.com/anbuchelva/image/upload/f_auto,q_auto/v1546629705/images/power-bi/Power_BI_Essbase_Initial_Experiments_8.png)
![Power_BI_Essbase_Connector_Initial_Experiments](https://res.cloudinary.com/anbuchelva/image/upload/f_auto,q_auto/v1546629705/images/power-bi/Power_BI_Essbase_Initial_Experiments_9.png)
![Power_BI_Essbase_Connector_Initial_Experiments](https://res.cloudinary.com/anbuchelva/image/upload/f_auto,q_auto/v1546629705/images/power-bi/Power_BI_Essbase_Initial_Experiments_10.png)
![Power_BI_Essbase_Connector_Initial_Experiments](https://res.cloudinary.com/anbuchelva/image/upload/f_auto,q_auto/v1546629705/images/power-bi/Power_BI_Essbase_Initial_Experiments_11.png)
![Power_BI_Essbase_Connector_Initial_Experiments](https://res.cloudinary.com/anbuchelva/image/upload/f_auto,q_auto/v1546629705/images/power-bi/Power_BI_Essbase_Initial_Experiments_12.png)
![Power_BI_Essbase_Connector_Initial_Experiments](https://res.cloudinary.com/anbuchelva/image/upload/f_auto,q_auto/v1546629705/images/power-bi/Power_BI_Essbase_Initial_Experiments_13.png)
![Power_BI_Essbase_Connector_Initial_Experiments](https://res.cloudinary.com/anbuchelva/image/upload/f_auto,q_auto/v1546629705/images/power-bi/Power_BI_Essbase_Initial_Experiments_14.png)
![Power_BI_Essbase_Connector_Initial_Experiments](https://res.cloudinary.com/anbuchelva/image/upload/f_auto,q_auto/v1546629705/images/power-bi/Power_BI_Essbase_Initial_Experiments_15.png)
![Power_BI_Essbase_Connector_Initial_Experiments](https://res.cloudinary.com/anbuchelva/image/upload/f_auto,q_auto/v1546629705/images/power-bi/Power_BI_Essbase_Initial_Experiments_16.png)
![Power_BI_Essbase_Connector_Initial_Experiments](https://res.cloudinary.com/anbuchelva/image/upload/f_auto,q_auto/v1546629705/images/power-bi/Power_BI_Essbase_Initial_Experiments_17.png)
![Power_BI_Essbase_Connector_Initial_Experiments](https://res.cloudinary.com/anbuchelva/image/upload/f_auto,q_auto/v1546629705/images/power-bi/Power_BI_Essbase_Initial_Experiments_18.png)

If you notice the `Period` Dimension, the levels represented in Power BI is similar to the Generation that we call in Essbase.

Logs for reference : ~~<https://hastebin.com/riqujohufi.sql>~~

{{< alert info >}}
Update: Expecting a proper document from Microsoft on connecting essbase from Power BI. Refer the [Git issue here](https://github.com/MicrosoftDocs/powerbi-docs/issues/540).
{{< /alert >}}
