+++
author = "Ryan Van Antwerp"
title = "Data Science With Your Friends: Seeing the Forest"
date = "2020-07-12"
description = "The beginning of my journey into Discord bots, natural language processing, and messing with my friends."
tags = [
    "discord",
    "natural language processing",
    "sentiment analysis",
    "reinhold",
    "totallynot"
]
categories = [
    "Data Science With You Friends",
]
aliases = ["seeing-the-forest"]
images  = []
+++

Like many busy adults, I keep in touch with a close-knit group of friends through group chat. We meandered through many different chat mediums but eventually settled on Discord. Discord is great - it has robust mobile and desktop clients, it allows us to mention and notify specific groups or users, and most importantly to me, it has a rich developer API. 

My group chat friends are tech savvy - we initially started the chat to better facilitate gaming sessions. As we grew older and life got hectic with homeownership, significant others, and starting families, the content of our discussions changed. We spent less time arguing with each other over how awful of a person I become when we play Dota 2, and more time talking about whether it would be a good idea to refinance a mortgage. At the end of the day, our group chat became an environment of growth and learning. And what better way to learn than to experiment on each other?

My friend Jake was the first to [build a bot for our group chat](https://www.bitlog.com/2017/03/31/techniques-for-effectively-growing-small-programs/). His bot, [crbot](https://github.com/jakevoytko/crbot), is a simple call and response bot. It could be taught to recall text and images based on key phrases.

```
ryan: ?ping
crbot: pong
ryan: ?learn shruggie ¯\_(ツ)_/¯
crbot: Learned about shruggie
ryan: ?shruggie
crbot: ¯\_(ツ)_/¯
```

Our group chat accepted it as one of our own - at first we didn’t really get it, then we tried to break it, then [we used it as a tool to ruin friendships](https://www.bitlog.com/2017/05/31/my-friends-trolled-each-other-with-my-discord-bot-and-how-we-fixed-it/), and now we can’t really live without it (OK, we could, but think of all the memes we’d lose in the process!)

I wanted to create my own bot. I’ve always had a fascination with natural language processing and machine learning but didn’t have any data or use cases to practice - suddenly I’m presented with a corpus of text, cleanly organized by author, and covering a wide range of topics of discussion. The possibilities are endless, but I had to start somewhere. My first task was one of analysis: how happy are my friends?

# Mock Trial with J. Reinhold

I wanted to start small and write a small bot that would just stand there ([menacingly](https://www.youtube.com/watch?v=LPmzRa-sXQs)) and listen, performing sentiment analysis on every message. From this simple idea, [reinhold](https://github.com/drvan/reinhold/tree/e3b47c908282b9d86a2871fc3e239abeb11d4c72) was born. Reinhold uses the fantastic Node.js [sentiment](https://github.com/thisandagain/sentiment) library, which when passed a sequence of text, would return a score identifying how positive (happy, content) or negative (angry, upset) the sequence was. The results were recorded for analysis, with names redacted to protect the innocent:

```
# SELECT user, SUM(comparative) FROM message GROUP BY user ORDER BY SUM(comparative) DESC;
user        SUM(comparative)
----------  ----------------
User1       20.4226536037062
User2       6.86666666666667
...
User7       -0.7750360750360
User8       -0.7929292929292 
```

Reinhold was small, but impressive... for about a week until my friends started spamming intentionally positive or negative words, trying to get a ‘high score’ in whatever extreme they were aiming for. I expected this; my friends are inquisitive and have a hacker mindset. I knew that anything I introduced into this chat would be abused and since Reinhold was little more than a proof of concept, I decided to retire the bot (for now). However, the flames were stoked - I wanted to do more than just analyze text; I wanted to create it!

# Totally Not Robots
Generating text is tricky. In the most primitive approach, I could store messages in their entirety and repeat one at random when commanded. On the other end of the spectrum, I could train a machine learning model on every message said and, with some effort, produce a message that sounds correct. In the middle of these two extremes lie Markov chains. If you haven’t heard of Markov chains, [you’re missing out](https://filiph.github.io/markov/). For the unfamiliar, a Markov chain is essentially a state machine where the next state depends only on information available in the current state. In the context of text generators, Markov chains are used to determine each subsequent word by looking at the current word and seeing what word typically follows that based on a training corpus. This might make more sense with an example. Suppose I was trying to use Markov chains to generate a text sequence for my friend who’s obsessed with Super Smash Brothers Melee. Let’s call her Sabrina. Here is the entirety of Sabrina’s chat messages:

```
Sabrina: I’d like to play melee.
Sabrina: Does anyone want to play melee for money? I’ve never played before.
Sabrina: When I get home from work, I’m probably going to play melee.
Sabrina: I just finished playing a game with Ryan, I never want to play Dota again.
```

To begin generating a Markov chain text sequence, I’d start with a random starter word. In the above context, my options are [I’d, Does, When, I]. Suppose my random starter word is “I”. After picking a word, I’d choose the next word based on what words follow my chosen word. In the above corpus, the following words follow “I”: [get, just, never]. Since none of these words appear after “I” more often than the others, I would again pick one at random.

Let’s extend this example a little further. Now suppose our current word is play. Using the messages above, the word melee follows play 75% of the time. The remaining 25%, the word Dota follows play. If we were writing a bot to generate text based on this Markov chain, we would use a simple weighted random number that would pick melee 75% of the time and dota 25% of the time. This fundamental property of Markov chains is why they work for text generation; if Sabrina speaks frequently about Super Smash Brothers Melee, a Markov chain text generator trained on that Sabrina’s messages would generate text that frequently talks about Super Smash Brothers Melee.

I saw that Markov chains worked well trained against large corpuses like Twitter or books, but I wondered if I could read every message in our group chat, store it in a queryable database and generate Markov chains on the fly. After a few weeks of tinkering in Typescript, my second bot, [totallynot](https://github.com/drvan/totallynot), graced our chat with its presence.

This bot worked by storing every word (or sequence of words, depending on the Markov chain order) by every author along with whether the given word was the beginning or the end of a sentence. When commanded to, the bot would pick a random starter, and iterate through each word until it hit an ending word, and then it would print the result. Most results were near incoherent, but there were some hilarious, yet not quite grammatically <cite>correct gems[^1]</cite>:

```
totallynot: top level fox/falco players probably routinely play between 200-300 APM which is trash
totallynot: man, i think she got admitted to committing a tumblr-level mistake if they manage to make another opt-in app that's useless for tournament play
totallynot: i sleep light, i definitely need to dress like this guy
totallynot: or to provide players with SARPBC experience is exceedingly small and turned them into a Rectangle-shaped box, call mutators and make passive aggressive judgements about everyone's responses)
```

Totallynot was not without faults - each sequence generated required a database query per word. Since this was running on a Raspberry Pi 3 in my basement, it wouldn’t be unusual for a chain to take up to 20 seconds to generate. For some of my friends (particularly those that send less messages than the rest of us), totallynot would almost never produce anything sounding like them. There’s definitely room for improvement.

# Bad Code Isn’t Always Bad

You may have noticed that I didn’t dwell much on the code or implementation of these bots and the reason for that is simple: the code is bad. Not awful, but pretty bad - and that’s OK. We’ve all written bad code or made poor design decisions. When I finished my [capstone engineering project at TCNJ](https://engineering.tcnj.edu/2009/10/06/tcnj-solar-boat/), I looked at what I had contributed to my group’s project and thought “if I did this over, I would do it completely different”. That thought initially led me to believe I did a bad job (which according to some, was true), but the reality is that by persisting through a series of challenges, I had learned an incredible amount of wisdom during the course of the project. Software engineering is no different - as you grow in your career, you may cringe at code you wrote just months ago, but this simply means you’ve learned from your mistakes and you’ll write better code moving forward.

Writing bots for the sole purpose of my friend’s amusement has been an experience. My friends greeted my bots with a mix of excitement, disinterest, and sometimes frustration. I found myself feeling pressure to improve these bots. With other software I’ve written in the past, my users aren’t so close to me. If thousands of people hit a web app I’ve developed and one of them has a bad experience due to a bug I’ve introduced, it’s hard to feel empathetic - by and large, I’d consider my software a success. But when writing software for my friends, the bad experience isn’t tied to a nameless metric on a dashboard; my friend, who I’ve known for close to a decade is having a bad time, and I’m responsible! Not only that, but my friend can call me out on my bad code, in front of the rest of my friends. Raw, brutally honest feedback is nerve-racking, but I believe this pressure will ultimately lead me to become a better software engineer.

Software that handles text and language is complex. Language is cultural and dynamic. A phrase that meant something a decade ago may mean something completely different today. It’s hard to define correctness. If my bot produced a sequence of text that’s supposed to sound like my friend, how do I know how accurate it was? 

Overcoming these challenges is the basis of this <cite>blog series[^2]</cite>. I’m going to rewrite these bots, follow better engineering patterns, get lost in the depths of natural language processing, and hopefully better myself along the way.

[^1]: Special thanks to Bryce for backfilling the totallynot database with our entire chat history, which ended up producing some of the greatest Markov chain sequences we’ve ever seen.
[^2]: The title of this series is a play on the game [Golf With Your Friends](https://store.steampowered.com/app/431240/Golf_With_Your_Friends/) - my friends and I would often play this while discussing tech, politics, the meaning of life, etc. I’m terrible at it, but it’s a great game. You should get it.
