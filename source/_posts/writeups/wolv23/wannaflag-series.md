---
title: "WolvCTF 2023: WannaFlag Series"
date: 2023-03-17 00:00:00
categories:
- ctfs
- wolv23
- osint
tags: 
- osint
description: "Hunting down a flag-stealing ransomware group with the power of OSINT — from Ethereum breadcrumbs to TF2 workshop maps, this is my writeup for the WannaFlag series from WolvCTF 2023."
short_description: "Hunting down a flag-stealing ransomware group with the power of OSINT — from Ethereum breadcrumbs to TF2 workshop maps."
category_column: "wolv23/osint"
permalink: ctfs/wolv23/osint/wannaflag/
thumbnail: /asset/banner/banner-wannaflag.png
---

![WannaFlag Banner](/asset/wolv23/banner.svg)

### Intro

Over the last weekend, I played in WolvSec's second CTF iteration with [Project Sekai](https://sekai.team/) — [WolvCTF 2023](https://ctftime.org/event/1866). We placed first in the open division, and throughout the solving process I became intrigued by a specific series of challenges placed under the OSINT category: **WannaFlag**. Telling a story of a supposed ransomware group which had been terrorizing the CTF community for the past several months, these series of challenges offered an opportunity for players to track down this group's means of operation. Ultimately, the goal was to find WannaFlag's kingpin through all possible methods.

Project Sekai was the first to blood the entire series. Here was our thought process, notes, and conclusions.

---

{% challenge %}
title: "WannaFlag I: An Introduction"
level: h2
authors: dree
solvers: enscribe
files: '[image.png](/asset/wolv23/image.png)'
genre: osint
points: 188
description: |
    Welcome to WolvCTF's OSINT Category! We have a bunch of great OSINT lined up, assuming nothing goes wrong hahahhahahhahah but why would it?  
    For this challenge, find where the image was taken, and look at the Google Maps reviews!  
    **Note**: Flags can be found in standard format `wctf{...}` for ALL OSINT challenges   
{% endchallenge %}

We're first given a little bit of a warmup: find the location of the following object, and to view its Google Reviews:

![image.png](/asset/wolv23/image.png)

Simple task! These types of challenges, often called GEOINT (geospacial intelligence), can be trivial if there is a landmark object situated within the image — in this case, we have some public art resembling a black cube. We can use [Google Lens](https://lens.google) to identify it:

![lens.png](/asset/wolv23/lens.png)

Looks like Google's given us a hit: this is "The Cube," a public art installation in the University of Michigan. Let's take a look at the Google Reviews:

![reviews.png](/asset/wolv23/reviews.png)

This `netcat wanna-flag-i dot wolvctf dot io one three three seven` can be converted to a command: `nc wanna-flag-i.wolvctf.io 1337`. Let's connect to this server to see what it has to say:

{% ccb html:true terminal:true scrollable:true wrapped:true %}
<span class="meta prompt_">$</span> nc wanna-flag-i.wolvctf.io 1337
== proof-of-work: disabled ==
Good job finding the Cube! It's a favorite destination among UofM students!
Anyways here is the flag:
wctf{sp1n
Huh???? Where did the rest of the flag g
                       ______
                    .-"      "-.
                   /            \
       _          |              |          _
      ( \         |,  .-.  .-.  ,|         / )
       > "=._     | )(__/  \__)( |     _.=" <
      (_/"=._"=._ |/     /\     \| _.="_.="\_)
             "=._ (_     ^^     _)"_.="
                 "=\__|IIIIII|__/="")
                _.="| \IIIIII/ |"=._
      _     _.="_.="\          /"=._"=._     _
     ( \_.="_.="     `--------`     "=._"=._/ )
      > _.="                            "=._ <
     (_/                                    \_)
                       ______
                    .-"      "-.
                   /            \
       _          |              |          _
      ( \         |,  .-.  .-.  ,|         / )
       > "=._     | )(__/  \__)( |     _.=" <
      (_/"=._"=._ |/     /\     \| _.="_.="\_)
             "=._ (_     ^^     _)"_.="
                 "=\__|IIIIII|__/="")
                _.="| \IIIIII/ |"=._
      _     _.="_.="\          /"=._"=._     _
     ( \_.="_.="     `--------`     "=._"=._/ )
      > _.="                            "=._ <
     (_/                                    \_)
                       ______
                    .-"      "-.
                   /            \
       _          |              |          _
      ( \         |,  .-.  .-.  ,|         / )
       > "=._     | )(__/  \__)( |     _.=" <
      (_/"=._"=._ |/     /\     \| _.="_.="\_)
             "=._ (_     ^^     _)"_.="
                 "=\__|IIIIII|__/="")
                _.="| \IIIIII/ |"=._
      _     _.="_.="\          /"=._"=._     _
     ( \_.="_.="     `--------`     "=._"=._/ )
      > _.="                            "=._ <
     (_/                                    \_)
██╗    ██╗ █████╗ ███╗   ██╗███╗   ██╗ █████╗ ███████╗██╗      █████╗  ██████╗
██║    ██║██╔══██╗████╗  ██║████╗  ██║██╔══██╗██╔════╝██║     ██╔══██╗██╔════╝
██║ █╗ ██║███████║██╔██╗ ██║██╔██╗ ██║███████║█████╗  ██║     ███████║██║  ███╗
██║███╗██║██╔══██║██║╚██╗██║██║╚██╗██║██╔══██║██╔══╝  ██║     ██╔══██║██║   ██║
╚███╔███╔╝██║  ██║██║ ╚████║██║ ╚████║██║  ██║██║     ███████╗██║  ██║╚██████╔╝
 ╚══╝╚══╝ ╚═╝  ╚═╝╚═╝  ╚═══╝╚═╝  ╚═══╝╚═╝  ╚═╝╚═╝     ╚══════╝╚═╝  ╚═╝ ╚═════╝
HAHAHHAHAHHAHA Ohhhhh man what an easy CTF to pwn

And I mean also really?? At least make a geo-osint KIND of difficult
The CTF is HOSTED by UofM where else would that dumb cube be????

Oh man ok well organizers if you want your "challenge" back or flags or whatever send 500,000 Goerli here:
0x08f5AF98610aE4B93cD0A856682E6319bF1be8a6

Who knows maybe we'll take more flags if you don't pay in time >:)
#YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs
#YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs
#YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs #YourFlagsBelongToUs
                       ______
                    .-"      "-.
                   /            \
       _          |              |          _
      ( \         |,  .-.  .-.  ,|         / )
       > "=._     | )(__/  \__)( |     _.=" <
      (_/"=._"=._ |/     /\     \| _.="_.="\_)
             "=._ (_     ^^     _)"_.="
                 "=\__|IIIIII|__/="")
                _.="| \IIIIII/ |"=._
      _     _.="_.="\          /"=._"=._     _
     ( \_.="_.="     `--------`     "=._"=._/ )
      > _.="                            "=._ <
     (_/                                    \_)
                       ______
                    .-"      "-.
                   /            \
       _          |              |          _
      ( \         |,  .-.  .-.  ,|         / )
       > "=._     | )(__/  \__)( |     _.=" <
      (_/"=._"=._ |/     /\     \| _.="_.="\_)
             "=._ (_     ^^     _)"_.="
                 "=\__|IIIIII|__/="")
                _.="| \IIIIII/ |"=._
      _     _.="_.="\          /"=._"=._     _
     ( \_.="_.="     `--------`     "=._"=._/ )
      > _.="                            "=._ <
     (_/                                    \_)
                       ______
                    .-"      "-.
                   /            \
       _          |              |          _
      ( \         |,  .-.  .-.  ,|         / )
       > "=._     | )(__/  \__)( |     _.=" <
      (_/"=._"=._ |/     /\     \| _.="_.="\_)
             "=._ (_     ^^     _)"_.="
                 "=\__|IIIIII|__/="")
                _.="| \IIIIII/ |"=._
      _     _.="_.="\          /"=._"=._     _
     ( \_.="_.="     `--------`     "=._"=._/ )
      > _.="                            "=._ <
     (_/                                    \_)
██╗    ██╗ █████╗ ███╗   ██╗███╗   ██╗ █████╗ ███████╗██╗      █████╗  ██████╗
██║    ██║██╔══██╗████╗  ██║████╗  ██║██╔══██╗██╔════╝██║     ██╔══██╗██╔════╝
██║ █╗ ██║███████║██╔██╗ ██║██╔██╗ ██║███████║█████╗  ██║     ███████║██║  ███╗
██║███╗██║██╔══██║██║╚██╗██║██║╚██╗██║██╔══██║██╔══╝  ██║     ██╔══██║██║   ██║
╚███╔███╔╝██║  ██║██║ ╚████║██║ ╚████║██║  ██║██║     ███████╗██║  ██║╚██████╔╝
 ╚══╝╚══╝ ╚═╝  ╚═╝╚═╝  ╚═══╝╚═╝  ╚═══╝╚═╝  ╚═╝╚═╝     ╚══════╝╚═╝  ╚═╝ ╚═════╝
██╗    ██╗ █████╗ ███╗   ██╗███╗   ██╗ █████╗ ███████╗██╗      █████╗  ██████╗
██║    ██║██╔══██╗████╗  ██║████╗  ██║██╔══██╗██╔════╝██║     ██╔══██╗██╔════╝
██║ █╗ ██║███████║██╔██╗ ██║██╔██╗ ██║███████║█████╗  ██║     ███████║██║  ███╗
██║███╗██║██╔══██║██║╚██╗██║██║╚██╗██║██╔══██║██╔══╝  ██║     ██╔══██║██║   ██║
╚███╔███╔╝██║  ██║██║ ╚████║██║ ╚████║██║  ██║██║     ███████╗██║  ██║╚██████╔╝
 ╚══╝╚══╝ ╚═╝  ╚═╝╚═╝  ╚═══╝╚═╝  ╚═══╝╚═╝  ╚═╝╚═╝     ╚══════╝╚═╝  ╚═╝ ╚═════╝
██╗    ██╗ █████╗ ███╗   ██╗███╗   ██╗ █████╗ ███████╗██╗      █████╗  ██████╗
██║    ██║██╔══██╗████╗  ██║████╗  ██║██╔══██╗██╔════╝██║     ██╔══██╗██╔════╝
██║ █╗ ██║███████║██╔██╗ ██║██╔██╗ ██║███████║█████╗  ██║     ███████║██║  ███╗
██║███╗██║██╔══██║██║╚██╗██║██║╚██╗██║██╔══██║██╔══╝  ██║     ██╔══██║██║   ██║
╚███╔███╔╝██║  ██║██║ ╚████║██║ ╚████║██║  ██║██║     ███████╗██║  ██║╚██████╔╝
 ╚══╝╚══╝ ╚═╝  ╚═╝╚═╝  ╚═══╝╚═╝  ╚═══╝╚═╝  ╚═╝╚═╝     ╚══════╝╚═╝  ╚═╝ ╚═════╝
Also all of you John OSINTs on twitter need to leave us alone
{% endccb %}

Well... that was unexpected. Let's first digest what in the world just happened to this netcat:

- Our flag is cut off midway, and WannaFlag supposedly "pwns" the server
- We're told to pay 500,000 Goerli to the wallet `0x08f5AF98610aE4B93...`
- This hashtag `#YourFlagsBelongToUs` is spammed everywhere throughout the message
- The group tells "John OSINTs" to leave them alone on Twitter

There's a small connection here, but it's not immediately obvious. We can actually search for this hashtag on Twitter, revealing a tweet from none other than `@JohnOSINT_` himself:

{% twitter url:'twitter.com/JohnOSINT_/status/1634680651768270848' %}

{% twitter url:'https://twitter.com/JohnOSINT_/status/1634682918764429313' %}

A simple [base64 decode](https://www.base64decode.org/) of the exfiltrated string gives us our first flag: 

{% flag %}
**WannaFlag I: An Introduction**: `wctf{uhhh_wh3r3_d1d_4ll_0ur_fl4gs_g0?}`
{% endflag %}

---

{% challenge %}
title: "WannaFlag II: Payments"
level: h2
authors: dree
solvers:
- Legoclones --flag
- sahuang
genre: osint
points: 348
description: |
    Ok well.........................something may have gone wrong  
    WannaFlag's ransom demand is insane, there's no way we are paying that. Can you figure out which address the money is being funneled to?  
    From the ransom note: send 500,000 Goerli to `0x08f5AF98610aE4B93cD0A856682E6319bF1be8a6`
{% endchallenge %}

Our next step involves figuring out where this ransom money is being funneled to. We can use [Etherscan](https://etherscan.io/)'s [Goerli Testnet Explorer](https://goerli.etherscan.io/) for transactions involving the address `0x08f5AF98610aE4B93cD0A856682E6319bF1be8a6`:

![Goerli](/asset/wolv23/goerli.png)

We can see the transactions this account has made so far, but the only ones that are relevant would be the ones done before the competition started (since those would be part of the challenge creation process). These are the following transactions the account has made within this reasonable scope:

![Goerli 2](/asset/wolv23/goerli2.png)

Looks like there's something that stands out — although there are a lot of "IN" transactions, there's only one "OUT" transaction. This is the transaction that we're looking for, and it's a pretty big one being sent to `0xA01FD0...`. Let's follow the breadcrumb:

![Goerli 3](/asset/wolv23/goerli3.png)

From here, it seems as if the money is being distributed into several different accounts. Upon checking these transactions, though, these addresses actually just loop back the crypto into the original `0x08f5AF...` wallet. Here is an example of one of these "dummy" accounts, `0xc527ad...`; the second transaction on this list sends money back in a seeming "loop":

![Goerli 4](/asset/wolv23/goerli4.png)

However, one outlier amongst these "OUT" transactions exists: the address `0x80710E...`, which funnels these payments into three different accounts:

![Goerli 5](/asset/wolv23/goerli5.png)

Once again, two of these accounts are dummies, and will send money back into `0x08f5AF...`. However, one of these accounts, `0x64E69A...`, will send money to a completely new address, `0x79616B...`, where the trail ends:

![Goerli 6](/asset/wolv23/goerli6.png)

There's a suspicious "SELF" transaction on this address. Hovering over it, we can see that Etherscan believes there is a hidden message in the "Input Data" field:

![Goerli 7](/asset/wolv23/goerli7.png)

If we take a look at the transaction hash and click on the "More Details" button, we find the flag in the "Input Data" section when decoding it into UTF-8:

![Goerli 8](/asset/wolv23/goerli8.png)

If you're lost, I made a visualization of the transactions which took place, highlighting the breadcrumbs which eventually led to the flag:

![Transactions](/asset/wolv23/transactions.svg)

{% flag %}
**WannaFlag II: Payments**: `wctf{g1v3_m3_b4cK_mY_cRypT0!!11!}`
{% endflag %}

---

{% challenge %}
title: "WannaFlag III: Infiltration"
level: h2
authors: dree
solvers: enscribe
genre: osint
points: 318
description: |
    We have some solid leads so far. However, we need our flags back. Find a way to locate their communication and infiltrate their private ransom service, and submit the stolen flag we wanted to use for the first OSINT!  
    From outside intelligence, we know the group sometimes goes by `w4nn4_fl4g`
{% endchallenge %}

We're now given a keyphrase to work with: `w4nn4_fl4g`. We can search for the specific term on Google by wrapping it in quotes, and our first result is a subreddit, [r/w4nn4_fl4g](https://www.reddit.com/r/w4nn4_fl4g/):

![Reddit](/asset/wolv23/reddit.png)

Looking through the small amount of posts on this locked subreddit, we can find three users in particular which have access to post permissions (alongside one moderator): `u/w4nn4fl4g_admin`, `u/RemarkableDiamond443`, and `u/[deleted]`. None of the posts and memes were particularly interesting or relevant, but one thing stuck out in particular: the deleted user. We can find the username of the user through querying [camas.unddit.com](https://camas.unddit.com/) for a specific comment in `r/w4nn4_fl4g`; let's utilize the comment "sorry" they left under [this post](https://www.reddit.com/r/w4nn4_fl4g/comments/11p6utf/questions/):

![First Query](/asset/wolv23/query1.png)

We found the original username of the deleted user: `u/Chemical_Bread1558`! We can now query for posts the user made under the subreddit:

![Second Query](/asset/wolv23/query2.png)

We've got a hit on a [moderator-deleted post](https://reddit.com/11p5w72). Let's check it out:

![Deleted Post](/asset/wolv23/deleted-post.png)

Well, that doesn't really help. However, we can see the post's original content by using the Unddit tool once again — simply replace the `reddit` in the URL with `unddit` to see deleted comments:

![Unddit](/asset/wolv23/unddit.png)

The secret [website](wanna-flag-d60bf7cd-012a-4fcc-9a4c-e60eca6b653f-tlejfksioa-ul.a.run.app) provided leads us to this:

![Website](/asset/wolv23/website.png)

We've managed to recover the flag from the first OSINT challenge, but it's actually meant to be submitted as part of the third:

{% flag %}
**WannaFlag III: Infiltration**: `wctf{sp1nnnNn_tH3_cUb333e3E}`
{% endflag %}

---

{% challenge %}
title: "WannaFlag IV: Exfiltration"
level: h2
authors: dree
solvers: enscribe
genre: osint
points: 491
description: |
    Now that we've successfully gotten into their website - I say we figure out what other data they have.  
    Find and crack the master flag list, and submit the flag you see of ours on the list.
{% endchallenge %}

<span style="color:#FF9999">**First blood!**</span> We're now tasked with analyzing the contents of their website. Scouring through the source code, we find two extra pages: `/ctfs.html` and `/prices.html`:

![CTFs](/asset/wolv23/website-ctfs.png)

![Prices](/asset/wolv23/website-prices.png)

We've also got a footer with various social medias (which are all unsurprisingly rickrolls):

![Footer](/asset/wolv23/website-footer.png)

However, on the `/ctfs.html` page, upon further inspection of the source code we find a secret page, `/data.html`, linked under the Instagram icon:

```html ctfs.html
      <footer class="w3-center w3-black w3-padding-64 w3-opacity w3-hover-opacity-off">
         <button onclick="topFunction()" class="w3-button w3-light-grey" title="Go to top"><i class="fa fa-arrow-up w3-margin-right w3-light-grey"></i>To Top</button>
         <div class="w3-xlarge w3-section">
            <!-- small logo with redirecting link -->
            <a href="https://youtu.be/dQw4w9WgXcQ" <i class="fa fa-twitter w3-hover-opacity"></a>
            <a href="data.html" <i class="fa fa-instagram w3-hover-opacity"></a>
            <a href="https://youtu.be/dQw4w9WgXcQ" <i class="fa fa-github w3-hover-opacity"></a>
         </div>
         <!-- page link -->
         </p>
      </footer>
```

Accessing the secret page reveals a secret download link:

![Data](/asset/wolv23/website-data.png)

Downloading the file, we find a `flaglist.xlsx`, a Microsoft Excel document. There is an issue, however — the spreadsheet is completely encrypted via password protection, and we cannot import it to Google Sheets:

![Fail](/asset/wolv23/fail.png)

Let's see what we can do about this. Inspecting the source code of `/data.html`, we actually find that the file is being sourced from a GitHub account, `fl4gpwners`:

```html data.html
      <h1 class="left topp">If you made it to this page, you must be an 3l1t3 h4x0r that's one of us. Nobody else wouldve sifted through those rickrolls. The stolen flag list is below. </h1>
      <a href="https://raw.githubusercontent.com/fl4gpwners/flaglist/main/flaglist.xlsx" class="button leftmar ">List of Flags</a>
      <p>      
      It's encrypted so you still need the password duhhh
      </p>
```

![GitHub](/asset/wolv23/github.png)

There happens to be an extraordinarily convenient `password-repo` repository on this GitHub account, which reveals a `passwords_stub.lst` file:

![Password Repo](/asset/wolv23/password-repo.png)

Unfortunately, none of the passwords in this list were accepted by the spreadsheet. However, it reveals extraordinarily useful information: the password list is a stub (as hinted by part of the filename), and the real password of the spreadsheet was likely part of the original document.

Let's further analyze the passwords which were given to us. We have the name of a U.S. city/town, followed by a number of some kind. Googling the city followed by the number reveals that the number is actually the population of the city which proceeds it; this means that `Concord_city` has a population of `105186`, `Baton_Rouge_city` has a population of `225128`, etc. We can use this information to our advantage to reassemble the password list, assuming it is every U.S. city/town followed by its population.

Upon further research, the population of the city was discovered to be sourced from the U.S. 2020 Census. [Census.gov](https://www.census.gov/data/tables/time-series/demo/popest/2020s-total-cities-and-towns.html), their official website, has various datasets of "Annual Estimates of the Resident Population for Incorporated Places of 50,000 or More." Importing the data into Google Sheets, we can see that this was the exact dataset which was used to generate the stub:

![Census](/asset/wolv23/census.png)

{% info %}
**Note**: See how the term "city", "munincipality", and "village" were lowercase in `password_stub.lst`? This is because the census uses these terms to categorize urban areas in the United States, and are not actually part of the area's name. That is also why there are some entries which have two "cities" in their name (e.g. `Oklahoma City city`), as city is both part of the official city name and categorization.
{% endinfo %}

Let's create a wordlist from this data. Although we can export it as a `.csv` and run a Python script to filter out the passwords, we can simply create a spreadsheet formula on another column to create the wordlist:

{% ccb caption:'Spreadsheet formula' lang:py %}
=SUBSTITUTE(SUBSTITUTE(LEFT(B5,FIND(",",B5)-1)," ","_"),",","")&"_"&SUBSTITUTE(C5,",","")
{% endccb %}

![Wordlist](/asset/wolv23/wordlist.png)

We can now copy the column into a text file, `wordlist.txt`, and use it to crack the spreadsheet. We'll be utilizing the `john` tool to achieve this.

Firstly, we need the hash of the spreadsheet. We can use the [`office2john.py`](https://github.com/openwall/john/blob/bleeding-jumbo/run/office2john.py) tool to extract the proper hash:

{% ccb html:true terminal:true %}
<span class="meta prompt_">$</span> python3 office2john.py /mnt/c/users/jason/downloads/flaglist.xlsx
flaglist.xlsx:$office$*2007*20*128*16*fc1b72a9c6a0f44e944534547b08d387*9c6e0ab272092ccd21181da4f3465dfd*223ecfbb03444d6a78149e26196e427bc448b5f6
{% endccb %}

After we pipe it into a file, `hash.txt`, we can now finally run `john`:

{% ccb html:true terminal:true highlight:8 %}
<span class="meta prompt_">$</span> ./john --wordlist=wordlist.txt hash.txt
Using default input encoding: UTF-8
Loaded 1 password hash (Office, 2007/2010/2013 [SHA1 256/256 AVX2 8x / SHA512 256/256 AVX2 4x AES])
Cost 1 (MS Office version) is 2007 for all loaded hashes
Cost 2 (iteration count) is 50000 for all loaded hashes
Will run 12 OpenMP threads
Press 'q' or Ctrl-C to abort, 'h' for help, almost any other key for status
Great_Falls_city_60506 (flaglist.xlsx)
1g 0:00:00:00 DONE (2023-03-19 15:35) 7.692g/s 5169p/s 5169c/s 5169C/s Palatine_village_67771..Hanford_city_57932
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
{% endccb %}

{% info %}
**Note**: For some reason, `john` when installed with `apt-get` doesn't have Office support (at least for me). Make sure that you are compiling from source if you want to use this tool (or just use `hashcat`)!
{% endinfo %}

The password is `Great_Falls_city_60506`, which we can now use to open the spreadsheet:

![Flaglist](/asset/wolv23/flaglist.png)

{% flag %}
**WannaFlag IV: Exfiltration**: `wctf{y0ur_fl4gS_b3l0nG_t0_m3_;)}`
{% endflag %}

---

{% challenge %}
title: "WannaFlag V: The Mastermind"
level: h2
authors: dree
solvers:
- Battlemonger --flag
- enscribe
- sahuang
- Violin
genre: osint
points: 500
description: |
    Alright, I don't know about you but I'm kinda sick of this WannaFlag group. I say we take them down once and for all. Maybe there's a way to figure out who is behind the whole operation...  
    Consider all possible leads and clues so far. This challenge may be the most complex so far.  
    No games or programs need to be downloaded, or users messaged.
hints:
- Perhaps we can find the Mastermind's email...
{% endchallenge %}

This challenge remained unsolved for a long while until the first hint was released for the challenge:

> Perhaps we can find the Mastermind's email...

This immediately gave us a starting path to work with. We can find the email address of the user from the commit history of `fl4gpwners`; I found it from the [patch](https://github.com/fl4gpwners/flaglist/commit/9c689f4c2e0582d20577a951bf72ae243d65146a.patch) of the commit which uploaded `flaglist.xlsx` to the `flaglist` repository:

![Patch](/asset/wolv23/patch.png)

We've found a `civilianengi3421@proton.me` — from here, although various email OSINT strategies yielded no results (e.g. [Epieos](https://epieos.com/)), a simple keyword search on Google resulted in a [GameBanana](https://gamebanana.com/members/2530374) profile:

![GameBanana](/asset/wolv23/gamebanana.png)

Let's take a look at this user. They currently have one submission, a map titled [`jump_forklift`](https://gamebanana.com/wips/74502) for the game [Team Fortress 2](https://www.teamfortress.com/):

![GameBanana Submission](/asset/wolv23/gamebanana-submission.png)

Check out that screenshot: it has a snippet of a Discord link, `discord.gg/aY4Wuy...`. It seems cut off, however, so we'll have to try and recover the rest of the invite.

Although we actually attempted to brute force the invite code (simply a two-character combination of `A-Z, a-z, 0-9`), we ended up completely IP rate-limited by Discord. So, like any logical person would do, I tried to open the map in the game itself.

To download maps into TF2, you need to subscribe to its respective workshop on Steam. Although GameBanana never explicitly provided the Steam account for this user (or so I believe), their Steam account conveniently had the same as their GameBanana, `civilianengi3421`:

![Steam](/asset/wolv23/steam.png)

Here is the [workshop](https://steamcommunity.com/sharedfiles/filedetails/?id=2945817953) item associated with `jump_forklift`:

![Workshop](/asset/wolv23/workshop.png)

After subscribing to the item, I booted up TF2 for the first time in a couple of years to check out what was going on. 

{% info %}
**Note**: The challenge explicitly states that you **do not** need to download any games or programs. I just simply took the easy route and did so, anyways!
{% endinfo %}

We can navigate to the "Create Server" menu and select the map at the bottom:

![Forklift](/asset/wolv23/forklift.png)

I entered the map and lo and behold, the Discord invite was fully visible:

![Discord Link](/asset/wolv23/discord-link.png)

Let's join the server... or not, I guess:

![Invalid](/asset/wolv23/invalid.png)

Although this hiccup had my team scratching their heads for a while, we eventually stumbled upon a discrepancy in the invite link presented in the screenshot and the one in the map — the screenshot's initial characters are `aY4Wuy...`, while the map has `aYWuyn...`. Let's try adding the missing character `4` to the invite link:

![Valid](/asset/wolv23/valid.png)

We've successfully gained access to the server! Let's take a look around:

![Discord](/asset/wolv23/discord.png)

Although there's nothing of relevance in any of the channels, we see that the server has two individuals who have interacted with each other: `rocketjumper3005` and `s0llym41n3006`, who had left the server earlier. Let's run a Sherlock search on these two users and see what we can find:

{% ccb terminal:true html:true %}
<span class="meta prompt_">$</span> python3 sherlock.py rocketjumper3005
[<span style="color:#277FFF"><b>*</b></span>] Checking username rocketjumper3005 on:

[<span style="color:#47D4B9"><b>+</b></span>] Coil: https://coil.com/u/rocketjumper3005

[<span style="color:#277FFF"><b>*</b></span>] Search completed with 1 results
{% endccb %}

{% ccb terminal:true html:true %}
<span class="meta prompt_">$</span> python3 sherlock.py s0llym41n3006
[<span style="color:#277FFF"><b>*</b></span>] Checking username s0llym41n3006 on:

[<span style="color:#47D4B9"><b>+</b></span>] Coil: https://coil.com/u/s0llym41n3006
[<span style="color:#47D4B9"><b>+</b></span>] Pastebin: https://pastebin.com/u/s0llym41n3006

[<span style="color:#277FFF"><b>*</b></span>] Search completed with 2 results
{% endccb %}

Coil was a false-positive, but that Pastebin account for the second user was a hit. Visiting their account reveals an interesting [paste](https://pastebin.com/GYJvaUNe):

![Paste](/asset/wolv23/paste.png)

The paste reveals that the `rocketjumper3005` user had been keeping their schoolwork in the Discord server, and it had been visible using BetterDiscord's [ShowHiddenChannels](https://github.com/JustOptimize/return-ShowHiddenChannels) plugin. The following image was attached:

![Screenshot](/asset/wolv23/screenshot.png)

We're given a couple hints to pick at: `rocketjumper3005`'s real name is Corey, and he had been working on his application to "TNISO University." Let's run a Google search for "corey tniso university" on DuckDuckGo:

![TNISO Search](/asset/wolv23/tniso-search.png)

We've got him! This "Corey Jacobs" actually has a [LinkedIn](https://www.linkedin.com/in/corey-jacobs-27259826a/) profile:

![Corey](/asset/wolv23/corey.png)

Expanding the "About" section reveals... some interesting text:

![About](/asset/wolv23/corey2.png)

A [base64 decode](https://www.base64decode.org/) reveals our final flag:

{% flag %}
**WannaFlag V: The Mastermind**: `wctf{y0u_c4n_r0ck3tjUmp_bUt_y0u_c4nt_h1d3}`
{% endflag %}

---

### Afterword

This was an extraordinarily well-designed challenge. A lot of OSINT nowadays isn't creative at all, and doesn't employ any sort of "out-of-the-box" thinking. The WannaFlag series, however, was my breath of fresh air — it brought in some really wacky and unique stuff, like the TF2 map/Steam (the Excel password cracking bit was more forensics, but that's just part of the nature of OSINT in general). I hope to see more of these types of challenges in the future. Here is a compiled list of tools that I used throughout the challenge — I hope you find them useful:

- [Google Lens](https://lens.google/)
- [Goerli Testnet Explorer](https://goerli.etherscan.io/)
- [Unddit](https://camas.unddit.com/)
- [Sherlock.py](https://github.com/sherlock-project/sherlock)
- [Epieos](https://epieos.com/)
- [Wayback Machine](https://archive.org/)