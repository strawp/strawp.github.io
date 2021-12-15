## How I Got Santa To Call My Kids

I'd been meaning to create some kind of app or Tasker script to create the illusion that I could phone Santa, or have him phone me and check on everyone then run this while everyone was in the car so that it would come in over bluetooth. 

I figure this might be the last year where they all believe in Santa so I decided to put some effort into it finally. Here's how I did it.

### The Effect

I toyed with the idea of having a call include a series of responses voiced as Santa: "Yes", "No", "Goodbye" etc, and have these run based on various hardware button presses that I could be discretely using even with the phone in my pocket. The advantage of this is that kids can ask Santa simple questions. The downside is that this is going to sound quite robotic and I can see them asking something which isn't going to be covered by pre-programmed responses.

I decided instead to go for a scripted conversation between me and Santa which would just play out and that I could interact with. This would *sound* more natural and allow me to include little details which, as anyone who has done social engineering or performed close up magic will know, really help to increase believability. 

I social engineer people in my job quite frequently. A bunch of kids shouldn't be too hard, right?

### The Script

I created a script as follows:

```
Me: Hello?
Santa: Hello there Mr Wallace, this is Santa
M: Sorry, who?
S: Santa. Father Christmas
M: Sure. Ok, if you're Santa, what did I get for Christmas when I was 9?
S: Ah, yes. You can tell you work in security don't you? Ho ho ho! Very well, let me see... 1988... 1989... it was a skateboard. It had a picture of a snake on the bottom of it and the words "Snake, ratsy and roll"
M: Ok, ok you're Santa. How can I help you?
S: Your Elf has been telling me about how impressed he's been with everyone's behaviour since he's been.
M: Everyone's?!
S: Yes, everyone: [lists names of all the kids] have all been trying really hard. He is most impressed.
M: Wow!
S: Hopefully Elf hasn't been too much trouble. He can be a little cheeky at times.
M: No, nothing too bad.
S: Good, good. Well, I must go, Mrs Christmas has just baked a batch of mince pies.
M: Oh, by the way. Do you still prefer Sherry?
S: Oh God bless you, Iain! I'll have what you're having thank you very much!
M: Right you are. Nice to talk to you!
S: Yes, lovely to chat. Bye for now!
M: Bye!
```

I then recorded the Santa half of the conversation in my best Santa voice and included appropriate length pauses for my half of the script to respond.

### Emulating a Phone Call

So, at this point I could just play the audio and respond to it. This would fool those in the car who couldn't see the dashboard head unit, but probably not my eldest. He is quite used to looking between the seats to see what track is playing or who is calling on the centre dash display. To do this convincingly it needed to be a proper phone call. Enter VOIP!

I selected a VOIP provider (sipgate.co.uk) that was just going to provide me with SIP connection details and not a pre-packaged app (as useful as those kind of providers are). Then I needed a way of playing audio down the line in a way that would play nice with `cron`. I found [sipcmd2](https://github.com/guisousanunes/sipcmd2) which is designed to run in the command line and supports a basic scripting language to perform various actions.

Once I'd got my sipgate account authorised I tried out `sipcmd2` but couldn't resolve an issue with the recorded audio coming through all distorted. I looked for another SIP client to try and found [Twinkle](https://github.com/LubosD/twinkle), which is an old but still pretty decent Linux VOIP client which also supports a decent range of command line options, perfect for scripting. It also has its own event-based scripting feature, so you can run custom scripts when a call is answered, someone rings etc.

![Twinkle scripting settings](./santa/twinkle_scripts.png)

Twinkle does not seem to have any functionality for just playing audio down the line, however. More hacks required.

### Piping Audio Down the Line

This is not a new problem, and there is software available for Windows and Mac that can create a virtual audio cable between input and output such that you can send your own audio from an app into microphone input for a Zoom call or online game etc. I found [a great guide](https://www.youtube.com/watch?v=Goeucg7A9qE) on how to do the same thing relatively easily in Linux as well. Now, if I play any media on my Linux server, it gets piped into the default microphone input.

### Automating Santa

As previously mentioned, Twinkle supports a bunch of options when run via command line, and also even has its own fully console-based client, `twinkle-console`. When you call Twinkle via the command line and it's already running, any actions are carried out on the currently running Twinkle, not a new process. Twinkle was clearly build by someone who loves to script *everything* :)

Here's the `playsanta.sh` script, which runs when an outgoing call is answered:

```bash
#!/bin/bash

# Get the directory of the script so that the audio file always resolves to the right place
SCRIPT_DIR="$( cd -- "$( dirname -- "${BASH_SOURCE[0]}" )" &> /dev/null && pwd )"

# Play the santa audio using mpg123
DISPLAY=:0 /usr/bin/mpg123 "$SCRIPT_DIR/santacall.mp3" &>> /tmp/log

# Hang up
twinkle --cmd bye
```

Note that because this is going to be running via `cron`, you need `DISPLAY=:0` otherwise `mpg123` won't find the audio sink and will quit with an error.

Then `call.sh` is the script which `cron` calls to launch the call itself:

```bash
#!/bin/bash
twinkle --immediate --call [my mobile number]
```

So, how this works is:

1. `call.sh` is run (by `cron` or whatever)
2. The call comes through on my phone
3. When I answer, the audio of Santa starts playing and is piped down the line to my phone
4. I try and remember my lines
5. Audio finishes, Santa hangs up

All that is left is to make sure when the call comes through it shows "Santa" and we have a nice picture just to drill it home. 

![Santa's contact profile](santa/contact.png)

### Result

Just after I locked the front door, I executed

```
sleep 200; ./call.sh
```

and then drove all with all the family. The call came through perfectly, and they all lost their minds, including their mum who I hadn't told anything about what I had planned. **Success!**
