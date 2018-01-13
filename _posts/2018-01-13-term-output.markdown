---
layout: post
title:  "Run python from node and stream output"
date:   2018-01-13 15:00:34 +0000
categories: rpicon node pty nodepty terminal
---

My current side project is [RPiCon][rpiconi] - a desktop application to develop python for the raspberry pi.

The main feature is that you will be able to run code on your desktop/laptop before deploying to the pi taking advantage of a virtual GPIO. Since I already had decided to write it as an electron application I needed a way to run and interact with python code from [nodejs][node].

When deciding how to run the code I had two requirements:

1. Being able to start and stop the python application easily
2. Streaming the output so that I could display it to the user in real time

### The obvious choice

As part of node's out-of-the-box api they have something called [Child Process][nodechildprocess] which is well documented and there are plenty of tutorials for it. So I added this snippet to my app:

```javascript
const { spawn } = require("child_process");

var pyProcess = spawn("python", "PATHTOFILE.py");

pyProcess.stdout.setEncoding("utf8");
pyProcess.stdout.on("data", data => {
  console.log(data);
});

pyProcess.stdout.on("end", data => {
  console.log("Token " + token + ": closing connection.");
});
```

This seemed to do the trick until I realized that the "data" event was not being emitted as soon as the code was printing to stdout but it was being buffered. Although buffering is probably better for some use cases I wanted an IDE-like experience where the output feels like a terminal output.

### The unmaintained choice

While I'm sure there is a way to persuade the node child process to not buffer the output my research actually lead me to [pty.js][ptyjs] which addressed both my requirements. As an added bonus the code didn't have to change much:

```javascript
const { spawn } = require("pty.js");

var pyProcess = spawn("python", [scriptPath]);

pyProcess.on("data", data => {
  console.log(data);
});

pyProcess.on("exit", exitCode => {
  console.log("Exiting with code " + exitCode);
});
```

However, when jumping from macos to linux and back I noticed that the behavior when the process completed was not consistent. More specifically the "exit" event did not seem to emitted on macos. Soon I realized that in linux the event was also not getting triggered for the right reasons - when the python code finished it would throw an error and cause the "exit" event to be kicked off.

After tinkering for a while with the lib locally and going through its github issues I couldn't find a solution and to make matters worse the project seemed abandoned.

### Fork this!

Fortunately, a kind soul has a forked this project to [NodePty] and has actively maintained it. Funnily enough I found this fork because there is a [pending PR][pendingpr] on pty.js to declare it unmaintained and point to NodePty.

Everything I wanted just worked by switching the dependency and replacing the import with:

```javascript
const { spawn } = require("node-pty");
```

It's amazing how many amazing tools are being being built and maintained by the OSS community out there. These are just a fraction of the options I had to build what I want and that's just awesome!

[RPiConI]: {{ site.baseurl }}{% post_url 2017-12-17-rpicon-i %}
[Node]: https://nodejs.org/en/
[NodeChildProcess]: https://nodejs.org/api/child_process.html
[PtyJs]: https://github.com/chjj/pty.js/
[NodePty]: https://github.com/Tyriar/node-pty
[PendingPR]: https://github.com/chjj/pty.js/pull/191
