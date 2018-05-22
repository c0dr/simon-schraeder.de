---
title: "Debug React inside Visual Studio Code"
date: 2018-05-22T21:18:27+02:00
draft: false
---

I was always a little intruigued by the "Debugging" tab in Visual Studio Code, but didn't really investigate what it could do.
Probably debug some Python code, but not the JavaScript I usually work with.
Recently though when working on a new React App, I realized I was totally wrong.

It turns out it is actually quite easy to setup and allows debugging of client-side JavaScript applications. You can set breakpoints directly inside your code, which really is a blessing. Setting breakpoint inside transpiled/minified code in Chrome always seemed like hit-or-miss.

The only thing you need is a `launch.json` in the `.vscode` folder in the project.

For a simple React App using webpack, it could look like this:

```json
{
  "version": "0.2.0",
  "configurations": [{
    "name": "Chrome",
    "type": "chrome",
    "request": "launch",
    "url": "http://localhost:3000",
    "webRoot": "${workspaceRoot}/src",
    "sourceMapPathOverrides": {
      "webpack:///src/*": "${webRoot}/*"
    }
  }]
}
```

* The `url` property specifies the site Chrome opens. You can also specify a `file` if you do not have a webserver running.
* The `request` property specifies how Visual Studio Code should treat Google Chrome. `launch` launches a new instance of Google Chrome with a clean profile, wheras `attach` allows you to attach to a Chrome instance that has remote debugging enabled on port `9222`.
* The `webRoot` property specifies where the files are located.
* The `sourceMapPathOverrides` override the source maps. The key is an expression that gets checked and if matching is replaced with value.

And that's about it. Create the JSON file, click on the debug symbol, click on "Start" and you are set.

To set a breakpoint, simply go to your JavaScript file and click on the right dot next to the line number.
The code execution will stop when hitting the breakpoint and you'll be able in see the current variables etc.