_Photography is good fun. I took a photo of my 10 Windows earlier on but it turned out too big for my photo viewer. Apparently 2GB is too big. :(_

_https://drive.google.com/file/d/1y4sfIaUrAOK0wXiDZXiOI-q2SYs6M--g/view?usp=sharing_

_Alternate: https://mega.nz/file/R00hgCIa#e0gMZjsGI0cqw88GzbEzKhcijWGTEPQsst4QMfRlNqg_

_Dev: Tom_

Note: The link might be invalid by the time you came here.

This is a memory forensics challenge. So we can use memory forensics tools like [Volatility](https://github.com/volatilityfoundation/volatility) to scrap this humongous 2GB file.

Then first thing to do is recognize what kind of OS this snapshot is based on, we do `imageinfo` on this.

`python vol.py -f ../imagery.raw imageinfo`

![imageinfo](https://github.com/spitfirerxf/Houseplant-CTF-2020/blob/master/imagery/imageinfo.png?raw=true)

It's based on `Win10x64_15063` profile, and it's recognized so we're good to go. Some of you might having trouble getting `volatility` to work properly, I suggest you to just pull it from the git repository and then use the via `python vol.py` instead of installing `volatility` via `pip`.

After that we explore a bit to see what the user are using.

`python vol.py -f ../imagery.raw --profile=Win10x64_15063 pstree`

![pstree](https://github.com/spitfirerxf/Houseplant-CTF-2020/raw/master/imagery/pstree.png)

There are a lot of services, like usual Windows like `svchost`. But there is a Notepad opened. Huh, Notepad is not a service. We might take a look at it. `notepad` command in Volatility can only be used for Win7 or earlier, so that's a bad sign.
To see what might be typed inside the Win10 Notepad, we need to dump the memory based on notepad. 6076 is the PID so we gonna scrap that up!

`python vol.py -f ../imagery.raw --profile=Win10x64_15063 memdump -p 6076 --dump-dir=../output`

aaaand it's dumping huge chunk of file.

It's dumped (like me haha get it)(just kidding), now we need to search the flag. It's a whopping 267MB of raw memory data, and most of it are junk, so we need to be a little bit smarter in our search. Basic search of `strings` doesn't work. Hmmmm....

Several Googling later, there is a tutorial on fetching data in notepad memory dump using volatility, and it also mentioned that Notepad processed text in Little-endian, and that's why normal strings didn't help at all, because it fetches using big-endian by default.

After that I opened the file using HexEditor, a free hex editor GUI which I would always recommend rather than having terminal `hexedit`, pretty straight-forward to use.

I searched for the `rtcp` tag with Little-Endian search option and boom.

![searchresult](https://github.com/spitfirerxf/Houseplant-CTF-2020/raw/master/imagery/flag.png)

The flag is
`rtcp{camera_goes_click_brrrrrr^and^gives^photo}`
