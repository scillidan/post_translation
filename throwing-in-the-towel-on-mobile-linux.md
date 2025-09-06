# [Throwing in the towel on mobile Linux](https://drewdevault.com/2023/06/16/Mobile-linux-retrospective.html)

I have been tinkering with mobile Linux – a phrase I will use here to describe any Linux distribution other than Android running on a mobile device – as my daily driver since about 2019, when I first picked up the PinePhone. For about 3 years I have run mobile Linux as my daily driver on my phone, and as of a few weeks ago, I’ve thrown in the towel and switched to Android.

The distribution I ran for the most time is [postmarketOS](https://postmarketos.org/), which I was mostly quite happy with, running at times sxmo and Phosh. I switched to [UBports](https://ubports.com/en/) a couple of months ago. I have tried a variety of hardware platforms to support these efforts, namely:

- Pinephone (pmOS)
- Pinephone Pro (pmOS)
- Xiaomi Poco F1 (pmOS)
- Fairphone 4 (UBports)

I have returned to LineageOS as my daily driver and closed the book on mobile Linux for the time being. What put the final nails in the coffin was what I have been calling out as my main concern throughout my experience: reliability, particularly of the telephony components.

| Use-case | Importance | postmarketOS | UBports | LineageOS |
| --- | --- | --- | --- | --- |
| Basic system reliability | 5 | 2 | 4 | 5 |
| Mobile telephony | 5 | 3 | 3 | 5 |
| Hotspot | 4 | 5 | 3 | 5 |
| 2FA | 4 | 4 | 1 | 5 |
| Web browsing | 4 | 5 | 2 | 4 |
| Mobile banking | 4 | 1 | 1 | 5 |
| Bluetooth audio | 3 | 4 | 2 | 4 |
| Music player | 3 | 4 | 1 | 3 |
| Reading email | 3 | 1 | 3 | 4 |
| Navigation aid | 3 | 2 | 1 | 5 |
| Camera | 3 | 3 | 3 | 5 |
| Password manager | 3 | 5 | 1 | 1 |
| sysadmin | 3 | 5 | 2 | 3 |


**More on these use-cases and my experiences**  
```md
**Mobile banking**: only available through a proprietary vendor-provided Android app. Tried to get it working on Waydroid; did not work on pmOS and almost worked on UBports, but Waydroid is _very_ unreliable. Kind of shit but I don’t have any choice because my bank requires it for 2FA.

**Web browsing**: I can just run Firefox upstream on postmarketOS. Amazing! UBports cannot do this, and the available web browsers are not nearly as pleasant to use. I run Fennic on Android and it’s fine.

**Music player**: the music player on UBports is _extremely_ unreliable.

**Reading email**: This is not entirely pmOS’s fault; I could have used my main client, aerc, which is a testament to pmOS’s general utility, but it is a TUI that is uncomfortable to use on a touchscreen-only device.

**Password manager**: pmOS gets 5/5 because I could use the password manager I wrote myself, [himitsu](https://himitsustore.org/), out of the box. Non-critical use-case because I could just type passwords in manually on the rare occasion I need to use one.

**sysadmin**: stuff like being able to SSH into my production boxes from anywhere to troubleshoot stuff.
```

Among these use-cases, there is one that absolutely cannot be budged on: mobile telephony. My phone is a critical communication device and I need to be able to depend on calls and SMS at all times, therefore the first two rows need to score 4 or 5 before the platform is suitable for my use. I remember struggling with postmarketOS while I was sick with a terrible throat infection – and I could not call my doctor. Not cool.

I really like these projects and I love the work that’s going into them. postmarketOS in particular: being able to run the same environment I run everywhere else, Alpine Linux, on my phone, is fucking amazing. The experience is impressively complete in many respects, all kinds of things, including things I didn’t expect to work well, work great. In the mobile Linux space I think it’s the most compelling option right now.

But pmOS really suffers from reliability issues – both on edge and on stable it seemed like every update broke some things and fixed others, so only a subset of these cool features was working well at any given moment. The breakage would often be minor nuisances, such as the media controls on my bluetooth headphones breaking in one update and being fixed in the next, or major showstoppers such as broken phone calls, SMS, or, in one case, all of my icons disappearing from the UI (with no fallback in most cases, leaving me navigating the UI blind).

So I tried UBports instead, and despite the general lack of good auxiliary features compared to pmOS, the core telephony was more reliable – for a while. But once issues started to appear, particularly around SMS, I could not tolerate it for long in view of the general uselessness of the OS for anything else. I finally gave it up and installed LineageOS.

Mobile Linux is very cool and the community has made tremendous, unprecedented progress towards realizing its potential, and the forward momentum is still strong. I’m excited to see it continue to improve. But I think that before anyone can be expected to use this as a daily driver, the community really needs to batten down the hatches and focus on one thing and one thing only: always, _always_ being usable as a phone. I’ll be back once more reliability is in place.
