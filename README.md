# pitagoras3.github.io

Sources for my small programming blog available on [pitagoras3.github.io](https://pitagoras3.github.io). 

This blog is based on [Hugo framework](https://gohugo.io) using [etch](https://github.com/LukasJoswiak/etch) theme.

To publish new post on blog just push commit to main branch. Flow of deployment is:

```mermaid
flowchart LR
    id1([main branch]) --> id2([GitHub action]) --static pages--> id3([gh-pages branch])
```

To run Hugo locally with fast render mode use command: 
```
hugo server
```