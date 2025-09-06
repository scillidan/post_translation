# [Why Web Apps Disappoint Me](https://entropicthoughts.com/why-web-apps-disappoint-me)

I've several times heard a variation of the following quote.

> One of the major ideas with HTML 5 is that it makes client-side software superfluous.

This is a distortion of what the point of HTML in general, and HTML 5 specifically, is made for. It's also hinting at web apps somehow being superior to native apps, which is fundamentally incorrect (although not always so in practise).

## What is HTML?

HTML is a language to mark up documents. This means that HTML is a way for an author to instruct the reader of the document that "this should be read as a heading, this should be read as body text, with emphasis here and here. The bottom thing here is a reference to another document, and at the top there is an image."

"The reader" in this case can refer to anything that tries to interpret the structure of the document – be it a human, a web browser, a screen reader or a document conversion program.

Now, HTML _5_ is fundamentally no different. However, it allows the author to be more specific when marking up the document. The author can now also say things like, "This section of the document is dedicated to aid in navigation of the page, this section is the footer of the page, and this section here contains a video." When the author can be more precise, the reader's job becomes easier – especially in the cases where the reader is a machine; a program converting an HTML page to a PDF document might skip trying to include a video.

But we're still only talking about instructions on how to read a document. HTML isn't really more than that.

Now that you know what kind of thing HTML is, you probably have a feel for what kind of thing the web was meant to do. That's one reason HTML based web apps will not be a decent replacement for real applications.

## Web Apps

There's a more fundamental reason: it increases the work you have to do on the technology stack involved. When you make web applications, you are essentially pretending the web browser is an operating system. Unfortunately, web browsers make terrible operating systems. Why is the web browser a bad operating system?

- It supports only one programming language, which was _not_ designed to create big applications
- It has limited support for drawing graphics other than through a combination of CSS and HTML, which are not the greatest design tools[^1]
- It has limited support for window management and other "desktop enviroment" tasks

Note well that these are all problems operating systems had a few _decades_ ago. We're just now re-living them inside the browser. Imagine if 20 years from now people are creating web browser web apps, and re-living the same problems _again_!

There is much lost with this additional layer of indirection. It's a price we should carefully consider before paying. However, there is also something to gain.

## Why Web Apps?

Web apps do have a significant advantage though, which is illustrated by the opening quote. Web browsers in practise work as a (somewhat) safe sandbox. Users dislike the churn of installing applications locally, having to deal with the operating system.

What in principle happens when you open a web app is that your browser

1. Downloads the entire application (or fetches it from the nearest cache)
2. Runs it in a sandbox, and when you are done
3. Deletes it (as far as the user is aware – it might still reside in a cache along the way)

If you asked 20 years ago how users would like to use applications in 2014, nobody would have predicted this. The challenge now lies in realising this development and supporting the "disposable application" workflow in the operating system. If operating systems supported this, people wouldn't turn to web browsers to do the same thing (because web browsers have other limitations the operating system does not have.)

I believe we're starting to see a shift toward this in the adoption of the "app stores" from handheld devices to desktop operating systems, but I also think there is still another leap of faith required to complete the transition.

[^1]: I know the canvas element exists and I know it allows drawing of arbitrary graphics and it can even be hardware accelerated and all that but it's still a very primitive tool. I've yet to see a GUI toolkit for canvas graphics. (Please, don't get any ideas.)
