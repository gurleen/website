---
title: On software simplicity and blogs
date: 2022-01-31
---

> "Inside every large program, there is a small program trying to get out."
> --C.A.R. Hoare

I think that blog software is overrated.

Unless you're running a site with many authors and many, _many_ readers, chances are you're not taking advantage of most of the features in a software package like Wordpress. Most blog posts (like this one) are easily contained in a Markdown file (like this one) and don't even need features like comments. In fact, this blog runs without a "dynamic" backend, frontend JavaScript, or even a static site generator. It's little more than a file server capable of templating and a handful of Markdown files.

_Quick aside: [John Gruber](https://daringfireball.net/), a co-creator of Markdown, is a Drexel University grad, which I think is pretty neat._

**Fantastic Files and Where To Find Them**

If you've spent any amount of time in the last two decades creating things for the World Wide Web from scratch, it's likely you've run into _Apache_ and _nginx_, the two most popular HTTP servers. They're fantastic pieces of software. They were created a long time ago, when the prevailing method of generating dynamic content for the Web was through CGI (Common Gateway Interface). This interface allowed web servers to execute a program before serving a document back to the user, which meant you could do things like accessing databases and setting off long-running tasks.

This is exactly how every PHP site works. Most PHP frameworks don't require that you run an application server like you might with a Python or JavaScript based backend. Every PHP file is, ostensibly, a script that runs when a request comes in and hydrates a document with data. This means that you can fill your `/var/www/html` directory with PHP files and `nginx` will happily serve them, handling dynamic data along the way, without much fuss.

Unfortunately, PHP has fallen out of popularity, and I think it's a darn shame. Not necessarily because I think PHP is a great language, but because I think we lost sight of a great principle:

> `simplicity == maintainability == happiness`

I find this to be a problem on a lot of corners of the Web. Websites that ought to be nothing more than a file server instead are bloated with loads of JavaScript, backends and databases that take ages to produce a result, and style decisions that leave your eyes weary. This website, for instance, is intended to be nothing more than a digital business card and a place for me to post my ~~deepest darkest secrets~~ thoughts. Why should it take more than a hundred milliseconds to load? Why add a ton of JavaScript to solve minor problems? _Why even have a backend?_

**Enter Caddy**

[Caddy](https://caddyserver.com/) is a modern, open-source HTTP server written in Go that handles things like SSL certificates for you. It's config file format is modern and easy to understand. And it took me ~5 minutes from me provisioning the $5 Linode VPS hosting this site to having a `Hello, world!` page served up over HTTPS. It's batteries-included, which means you get goodies such as Markdown support and templating.

Why is this important? Because it lets us build a low-maintenance blog without writing any application code (at long as you don't consider the small bits of logic in our templates to be "application code"). The base file for this page, [`/blog/post/index.html`](https://github.com/gurleen/website/blob/main/blog/post/index.html), uses this small bit of Go template language before the markup to get every Markdown file and prepare them for display:

```html
{{$pathParts := splitList "/" .OriginalReq.URL.Path}} {{$markdownFilename :=
default "index" (slice $pathParts 3 | join "/")}} {{$markdownFilePath := printf
"/blog/posts/%s.md" $markdownFilename}} {{if not (fileExists
$markdownFilePath)}}{{httpError 404}}{{end}} {{$markdownFile := (include
$markdownFilePath | splitFrontMatter)}} {{$title := default $markdownFilename
$markdownFile.Meta.title}} {{$date := default "" $markdownFile.Meta.date}} ...

<div class="container spacer-above">
  <h2>{{ $title }}</h2>
  <h4>{{ $date | date "2006-01-02"}}</h4>
  {{markdown $markdownFile.Body}}
</div>
```

That's it. All I have to do now is to drop my Markdown files into `/blog/posts` and Caddy will render the Markdown as HTML, respecting the CSS I created for the website. There's no database, no build step, nothing. Just a clever use of a simple template language.

By the way, [the blog index](/blog/) also uses very little extension to the markup in order to display all of my blog posts:

```html
{{range $i, $file := (listFiles "/blog/posts")}}
    {{$markdownFilePath := printf "/blog/posts/%s" $file}}
    {{$markdownFile := (include $markdownFilePath | splitFrontMatter)}}
    {{$title := default $file $markdownFile.Meta.title}}
    {{$date := default "" $markdownFile.Meta.date}}
    <tr>
        <td>{{$date | date "2006-01-02"}}</td>
        <td>{{$title}}</td>
        <td><a class="button" href="/blog/post/{{($file | trimSuffix ".md")}}">Read</a></td>
    </tr>
{{end}}
```

**A Plea**

As a result of setting the site up in this way, pages load in well under a second. Because it's 2023. Computers are more powerful than ever, and things should not take a while to load. 

If you're building something for others to enjoy on the Web, I implore you to see if the tools you're using are useful or just add bulk. Of course there are instances where the extra complexity is warranted -- you're not building YouTube in this manner. But in a world in which everyone in the tech industry strives to use what's new and trendy, consider that you don't need it. Consider that there's a better way of doing things that don't involve ripping your hair out. Where you can understand everything that's happening, not hidden behind a sea of TypeScript compilers and NPM. 

If you think that this makes me sound like a tech curmudgeon, blame the biggest such person I know: [Dr. Brian Stuart](https://www.cs.drexel.edu/~bls96/). At the beginning of my undergraduate studies, he instilled in me this principle of simplicity. Of course, he'd insist that all the world's programs could be written in C, but I digress. It's about the principle.