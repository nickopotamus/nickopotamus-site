---
# Documentation: https://wowchemy.com/docs/managing-content/

title: "Automagically downloading PDFs from Sci-Hub with Zotero"
subtitle: "Yo ho fiddle de dee, being a pirate is all right with me. Within limits."
summary: "Zotero will try to download  PDF copies of anything you add to it... but what if it's stuck behind a paywall?"
authors: []
tags: ["nerdiness"]
categories: []
date: 2021-01-05T16:20:26Z
lastmod: 2021-01-05T16:20:26Z
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: ""
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: ["geekiness"]
---
[Zotero](http://zotero.org/) is clearly [the finest citation manager](https://subjectguides.library.american.edu/c.php?g=479020&p=3323781) on the market today. It's [free](https://www.zotero.org/getinvolved/), offers one click importing [via broswer plugins](https://www.zotero.org/download/connectors), manages webpages (and pretty much any other resource you can think of) in addition to papers and books, works offline, exports to [BibTex](http://www.bibtex.org/), and automagically downloads PDFs for anything you add. [It even searches Unpaywall for you](https://www.zotero.org/blog/improved-pdf-retrieval-with-unpaywall-integration/), but even then, sometimes those pesky PDFs are still [stuck behind paywalls](https://www.theguardian.com/education/2019/mar/28/paywalls-block-scientific-progress-research-should-be-open-to-everyone) and difficult to get to, even though you have [Athens](https://www.openathens.net/) or other institutional access to legally access them.

That's where [Sci-Hub](https://en.wikipedia.org/wiki/Sci-Hub) comes to youe slightly illegitimate rescue. [Alexandra Elbakyan](https://engineuring.wordpress.com/)'s website provides papers that would otherwise be stuck behind a paywall or registration only site. It has its [high profile detractors](https://www.theweek.in/news/india/2021/01/04/sci-hub-and-libgen-row-academics-speak-out-as-delhi-hc-set-to-decide-piracy-case.html), and obviously **I'm not advocating [academic piracy](https://www.enago.com/academy/academic-piracy-revolution-or-robbery/)** but if there was some way to integrate Sci-Hub with Zotero it would provide a very useful workaround to download PDFs with one click, _to which you do have legitimate access_, without the faff of remembering your login details, finding them, downloading them, then attaching them to the reference manager.

Well, you're in luck. Simply:

1) Open the Preferences
2) Move to the Advanced tab, and click "Config Editor" at the bottom. 
3) Accept the "you may break this warning"
4) Use the search box to find ```extensions.zotero.findPDFs.resolvers```
5) Right-click/double click and replace ```[]``` with:
```
{
    "name":"Sci-Hub",
    "method":"GET",
    "url":"https://sci-hub.se/{doi}",
    "mode":"html",
    "selector":"#pdf",
    "attribute":"src",
    "automatic":true
}
```
Job done. Now if Zotero can't automatically download the PDF, it'll search Sci-Hub for it, and download that instead. Remember, **doing this for papers you don't have legitimate access to is illegal**, but it's a nifty little time-saving workaround if you do.

![Lazy town knew it...](/post/zotero-sci-hub/pirate.gif)
