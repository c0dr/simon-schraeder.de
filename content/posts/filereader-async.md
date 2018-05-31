---
date: "2018-05-31T12:51:20+02:00"
title: "Using the FileReader API in async functions"
authors: []
categories:
  -
tags:
  -
draft: false
---

The FileReader API allows you to read files from the users' computer. Due to sandboxing of course this is mostly limited to files the user has selected in the file input fields.

The most interesting functionality is probably the `.readAsArrayBuffer()` function as it allows you to selectively read the file and only store part of it in memory. This allows client-side applications to process huge files without the need for huge memory.

While using `async` functions however I discovered that the `FileReader API` only supports callbacks. This is a bit annoying, as `async/await` allows much cleaner code.

The easiest solution for this is to write a helper function and wrap it into a Promise.

```js

function readFileAsync(file) {
  return new Promise((resolve, reject) => {
    let reader = new FileReader();

    reader.onload = () => {
      resolve(reader.result);
    };

    reader.onerror = reject;

    reader.readAsArrayBuffer(file);
  })
}

async function processFile() {
  try {
    let file = document.getElementById('fileInput').files[0];
    let contentBuffer = await readFileAsync(file);
    console.log(contentBuffer);
  } catch(err) {
    console.log(err);
  }
}

```

And that's it. The contentBuffer variable now contains the ArrayBuffer with no need for us to do anything more.
We're wrapping the async function content in a try/catch block to make sure we catch the rejection case.

To process the content of the file further we can simply use the `TextDecoder` API.
It allows us to convert the `ArrayBuffer` to a `String`. The best idea obviously is to only process parts of the `ArrayBuffer` as converting the whole might lead to memory issues.

A small example can be seen in the following jsfiddle:

<iframe width="100%" height="700" src="//jsfiddle.net/g3n8bzp7/1/embedded/js,html,result/" allowpaymentrequest allowfullscreen="allowfullscreen" frameborder="0"></iframe>

The first 20 bytes of the selected file are converted to UTF-8 and displayed in the `pre` below.
I think this is quite a powerful example for the clean code that `await/async` allows us to build.