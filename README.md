# pitagoras3.github.io

Sources for my small programming blog available on [pitagoras3.github.io](https://pitagoras3.github.io). 

This blog is based on [Hugo framework](https://gohugo.io) using [etch theme](https://github.com/LukasJoswiak/etch).

To publish new post on blog just push changes to `main` branch. After that, GitHub action builds static pages and commits them to `gh-pages` branch. From there, they're available via GitHub Pages on [pitagoras3.github.io](https://pitagoras3.github.io).

```mermaid
flowchart LR
    id1([main branch]) --> id2([GitHub action]) --static pages--> id3([gh-pages branch])
```

You can run this blog locally with Hugo using fast render mode by command: 
```
hugo server
```

___

###### Favicon licence

```
Twemoji graphics made by Twitter and other contributors, licensed under CC-BY 4.0: https://creativecommons.org/licenses/by/4.0/
```