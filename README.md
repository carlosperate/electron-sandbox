![](electron-sandbox.jpeg)

> This repository has not been reviewed for security flaws, use at your own risk.

# electron-sandbox
A simple example of a sandboxed renderer process with the ability to communicate to the backend (main.js) through IPC in a _secure_ manner.

If you're developing an electron application then I strongly recommend you to read the [Blackhat Electron Security Checklist](https://www.blackhat.com/docs/us-17/thursday/us-17-Carettoni-Electronegativity-A-Study-Of-Electron-Security-wp.pdf).

## preload-simple
A very simple pre-load script that serves as a dummy for tutorial purposes.

## preload-extended
A good implementation of a secure preload script, and main process initialization (main.js). 
You can use this version in production.

## usage

```
$ npm install electron
$ electron main.js
```

## Why?
If you're dealing with potentially untrusted content (displaying videos,images, text, etc..) in your Electron application, then you should run it with the sandbox enabled. The sandbox provided by Chrome is very strong, it has the ability to mitigate zeroday exploits within the Chrome browser engine (Blink) by restricting the ability of the attacker.

The sandbox is disabled by default in Electron (not in Chromium). Enabling the sandbox removes the ability to use NodeJS freely within the renderer process. Code that uses NodeJS should be moved from the renderer process to the main process. The sandboxed renderer process can then trigger the execution of these tasks (in the privileged browser process) and get the return values. The renderer process is restricted to the tasks that you allow it to execute, however a clever attacker could potentially use them to his/her benefit so construct them carefully. Give the renderer process the least amount of privilege, "make it dumb". 

Things renderer processes shouldn't be able to do:
* execute/create a new process
* freely read and write to whatever file they want
* freely pick a channel to send IPC messages over (use a whitefilter instead)

## Terminology
Electron is built on top of Chromium, a multi-process browser.  What's so important you might wonder, multi-process sounds really boring. Well you're probably right about that one. The reason for being a multi-process browser is a simple one: security. 

The idea behind it is equally simple, we split the weak from the strong. Code has bugs, and those bugs are sometimes exploitable. The thing is, as the complexity of code rises, the numbers of bugs rises too. The engine, that turns html, css & js into a visible thing on your screen, is one damn complex thing. 

_**rendering**_: turning code into what you see on your screen.

A lot of the complexity sits in the rendering engine, it's also the engine that is responsable for executing the JavaScript being loaded in the html files!
It's exactly this engine that's known to contain bugs. Have you ever been infected by visiting a website? The malicious entity probably found a way to exploit a bug in the rendering engine. 

You haven't forgetten the multi-process thing right? Well the chromium browser actually executes this vulnerable rendering engine in its own processes. Multiple renderer process co-exist on the same host. 

_**Renderer process**:_ the renderer processes evaluate and render the html, css and executes js.

![overview](https://www.chromium.org/developers/design-documents/site-isolation/ChromeSiteIsolationProject-arch.png?attredirects=0)

You might wonder, what executes the render processes? That's a good question, with a simple answer:
the _**browser process**_ or also known as the _**main process**_. You can kinda look at the browser/main process as the boss who makes sure his minions are working. 

Pro tip: Create two folders: 'render' and 'main'. This will help you keep track of which files are executed in the main process and which ones are executed in the renderer process.

Seperating the browser from the rendering engine actually doesn't do shit for security. Yeah I lied lol. Merely splitting it in multiple processes doesn't do crap for security. This is where the real magic happens.

Protecting against such bugs is easier if we maintain two levels of access. The renderer processes is a baby. Do you know where babies like to play? They play in **a sandbox**, well it's a magical sandbox. Any sandcastle you build is destroyed afterwards. You can't escape the sandbox (or atleast it's designed to be unescapable, but bugs happen..). Anything you do in the sandbox, stays in the sandbox. Technically speaking, the sandbox is a way to encapsulate existing code and execute it in the locked-down process with safeguards to prevent it from accessing or doing certain things. 

The browser/main process does *not* execute in such a sandbox, so it wields a lot more power than a renderer process. The renderer process is very limited in what it can do when it's sandboxed, but it does have the ability to communicate with the browser process over **Inter-Process Communication** or IPC for short. 

Electron is a bit of a different beast than Chromium, as it also provides you with a very powerful NodeJS API by default with your electron application. You might have already guessed it, the sandbox is disabled by default. 

This example will re-enable the sandbox and deal with the limitations. We get a chance to do _some_ of the NodeJS API in the renderer process. This is due to the **preload.js script**, it runs in a private scope  but be carefully what you leak to the global scope (ie: attaching functions to the window) and it has _some_ NodeJS functionality. We prefer having a more privileged environment, so we only use the preload script to act as a bridge. The bridge that connects the renderer process with the browser process: making available the **ipcRenderer**. Check out preload-simple.js or preload-extended.js for more information. 

### Good reads about Chromium & their sandbox:

http://blog.azimuthsecurity.com/2010/05/chrome-sandbox-part-1-of-3-overview.html

http://blog.azimuthsecurity.com/2010/08/chrome-sandbox-part-2-of-3-ipc.html

Don't bother looking for part 3 - the author never wrote it.

### Good reads about how the sandbox works with electron & NodeJS

A quite in-depth presentation about the security of electron and how it can be improved:

https://www.blackhat.com/docs/us-17/thursday/us-17-Carettoni-Electronegativity-A-Study-Of-Electron-Security.pdf

https://electron.atom.io/docs/api/sandbox-option/

https://www.blackhat.com/docs/us-17/thursday/us-17-Carettoni-Electronegativity-A-Study-Of-Electron-Security-wp.pdf

