# Metamask Firefox Fedora 39

BEFORE YOU PROCEED, BACK UP YOUR CURRENT INSTALLATION/PROFILE, MAKE A VM SNAPSHOT, ETC. BUT DO MAKE SURE YOU HAVE A WAY TO RESTORE AND CONTINUE IF YOU MAKE ANY MISTAKES.

Also keep in mind that downloaded files as well as local files and directories may have a different name so you must change it when running the commands (e.g. firefox bzip2 file, profile IDs, etc.)

Per the instructions on the linke bewlow:

https://support.metamask.io/hc/en-us/articles/360018766351-How-to-recover-your-Secret-Recovery-Phrase

If you get a scuttle/scuttling error like the one below or `runtime-lavamote.js` does not allow you to extract the json data string you may try the steps below to recover your metamask wallet and brute force the hash password.

`Uncaught Error: LavaMoat - property "chrome" of globalThis is inaccessible under scuttling mode. To learn more visit https://github.com/LavaMoat/LavaMoat/pull/360.`

First you have to get firefox developer edition as you need to disable scuttling on `runtime-javamoat.js` and recreate the XPI zip archive which then will not be signed by mozilla and regular/production firefox will not allow you to use the metamask extension and it will be disabled. For linux or recent fedora versions you can usually download the bzip2 archive from their site and extract it using `bunzip firefox.bz2` and then extract the tar file using `tar -xvf firefox.tar`.

Once you have installed/extracted the developer edition launch firefox and change the following settings in about:config by setting them to false

`xpinstall.signatures.required`

`dom.event.clipboardevents.enabled`

I did end up changing `xpinstall.whitelist.required` to false as well but may not be required

The original machine was fedora aarch64 and at the time of this writing firefox did not provides such binaries for linux so I ended up copying the `/home/user/.mozilla` directory to a x64_64 fedora machine.

You can then launch firefox from the command line with the -p option to point firefox developer edition to that profile by going to the extracted directory `cd Downloads/firefox` and running `./firefox -p .mozilla`, once you launch it you will be greeted by a dialog asking to load the profile, if you hover over the profiles it will give you the uuid of the profile and one way to verify which one to chose is by looking at the directory `.mozilla/firefox/a47sfyj0.default-release` and you will see that it's the same ID as the one where your wallet is saved, it can also show another default profile but you just need to make sure it's the one where metamask was installed.

When I copied the folder to the other machine it did not copy the wallet file and when I went through the instructions in the link I did not get to the wallet login screen but it was instead asking to add a wallet or create a new one, we will work around it in the steps below.

As I mentioned above in order to disable scuttling you need the XPI zip archive and edit runtime-lavamoat.js file, one easy way of finding the file is by running the command `find .mozilla/ -name *.xpi` which yielded:

`.mozilla/firefox/a47sfyj0.default-release/extensions/webextension@metamask.io.xpi`

I recommend creating an extract directory using `mkdir xpi` and copying the zip file one directory above, make sure you cd to the directory `cd xpi`.

Extract the file using `unzip ../webextension@metamask.io.xpi`, all the files will be extracted to your current directory `xpi`.

At this point you can edit the file `runtime-lavamoat.js` file by going to line 96 (as of this writing, keep in mind code updates may change the line numbers) where `scuttleGlobalThis` is defined, line 97 should look like the string below and you can disable scuttling by setting `"enabled":false`.

`{"scuttleGlobalThis":{"enabled":true,"scuttlerName":"SCUTTLER","exceptions":["toString","getComputedStyle","addEventListener","removeEventListen      er","ShadowRoot","HTMLElement","Element","pageXOffset","pageYOffset","visualViewport","Reflect","Set","Object","navigator","harden","console","WeakSet",      "Event","Image","/cdc_[a-zA-Z0-9]+_[a-zA-Z]+/iu","performance","parseFloat","innerWidth","innerHeight","Symbol","Math","DOMRect","Number","Array","crypt      o","Function","Uint8Array","String","Promise","JSON","Date","__SENTRY__","appState","extra","stateHooks","sentryHooks","sentry"]}}`

Save the file and with the oringal XPI zip file on the directory above you can run the command below to update/recreate the file, you can also use the 'a' option instead of 'u' but it doesn't matter, updating it does recreate the file and it will not be singed by firefox:

`7z u ../webextension@metamask.io.xpi * -r`

To install 7zip on fedora you can run:

`sudo dnf install p7zip-plugins`

At this point if you didn't need to move anything to a new machine you may continue with the instructions in the link above but in my case for some reason the wallet file was missing, I zipped the `.moziila` directory before I copied it over so I don't really know why it wasn't there. If you are my a similar situating remember that the link asks you to run some javascript code to access the vault so what I did was search for any files that contain the word 'vault' in the `.mozilla` directory by using `grep -rin vault .mozilla/` which then yielded:

`grep: .mozilla/firefox/a47sfyj0.default-release/storage/default/moz-extension+++61cec338-5b99-4c18-9f9f-c3f7fcf237ce^userContextId=4294967295/idb/3647222921wleabcEoxlt-eengsairo.files/430: binary file matches`

The actual file that we care about is named 430 in that directory and it is a snappy framed data format (the wallet itself), don't spend too much time trying to open it or decompress it because it gets a lot simpler from here (filewise, bruteforcing the hash is still the harder task).

Ultimately I copied the file named `430` into the `.mozilla/firefox/a47sfyj0.default-release/storage/default/moz-extension+++61cec338-5b99-4c18-9f9f-c3f7fcf237ce^userContextId=4294967295/idb/3647222921wleabcEoxlt-eengsairo.files/` directory on the other machine.

Once I had both the XPI zip and the wallet file on the new machine I was able to run firefox developer edition with signatures disabled and allowed copy paste on the console to continue to follow the instructions in the link.

Now, once you are able to get the json string with the hash data you can save the whole json string to a new file, for example `datafile.json` and it needs to be converted from metamask to hashcat format but you first need to clone the hashcat repository for the necessary python script.

`git clone https://github.com/matrix/hashcat.git`

`cd hashcat/tools/`

`./metamask2hashcat.py --vault /path/to/datafile.json`

This should ouput a new hash in hashcat format for meta mask starting with `$metamask$`

Copy the whole string to a new file, for exmaple `archivehash.txt`

I didn't build hashcat from the downloaded repository as I only needed the conversion tool and you can install hash cat in fedora using:

`sudo dnf install hashcat`

Now as fas as hascat, this is where you enter the challenge on your own as many things depend on how long it will take given how many GPUs you have access to plus the options and methods you chose or are best for your case.

As you can see on this link https://hashcat.net/forum/thread-10386.html the user is attempting a brute foce attack using `-a 3`, you can read through all the options by running `hashcat --help`.

`hashcat -a 3 -m 26600 -o output.txt hashes.txt -w 3 ?a?a?a?a?a?a?a?a?a?a?a`

He gets the error : `"Integer overflow detected in keyspace of mask: ?a?a?a?a?a?a?a?a?a?a"` because the mask used (string at the end) is incompatible as it may be too long of a string but it could also be due to some other issue.

You can learn more about masks here https://hashcat.net/wiki/doku.php?id=mask_attack

You can find a lot of documentation on the FAQ here https://hashcat.net/wiki/doku.php?id=frequently_asked_questions

Here are some of the iterations I played around with:

`hashcat -a 3 -m 26600 -o output.txt -d 1,2 -w 3`

`hashcat -a 6 -m 26600 -o output.txt -p \: -d 1,2 -w 3 archivehash.txt wordslist.txt`

`hashcat -a 6 -m 26600 -o output.txt  -d 1,2 -w 3 archivehash.txt wordslist.txt -1?u?l?d?u?l?d?u?l?d`

The paraemters above mean the following:

`-a 3` for bruteforce, `-a 6` for a combination attack like bruteforce with a words list dictionary, `-d` means the device to use for example 1 is the CPU and 2 is the GPU but you can try both by seaprating them by a comma, `-w 3` is the intensity so the higher you go the harder it will try and the more power it will consume. You need to install the CUDA toolkit as well and if you get an RTC lib error from hashcat it may not be a show stopper as it may help optimize the processing but it will then continue to use the GPU.

You can use tools like princeprocessor to generate a permutation of words list but if you know part of the password it creates excesive combinations of the words that aren't really similar to what the password used to be. Also you can create files with maks permutations as well but the code I ran that created the file ended up making all of them incompatible and kep getting the interger overflow error over and over so as I said above you will need to do some reading on both hashcat help as well as their documentation to better understand each approach.

I noticed that the following script here https://gist.github.com/rinchik/66702245a0f79ff4fbe731748269b275 can create a much better list but once you add all the uppercase letters, numbers and special characters it blows up memory in the system very fast so it's not efficient. I'm working on rewriting it so it saves and appends to a file every time and doesn't use lists or create a bunch of copies every time. This isn't really hard so I assume you can take on this task as well if you've made it this far. Having a words list that include parts of the original password will narrow the processing and can help compute/crack the hash faster.

Good Luck!
