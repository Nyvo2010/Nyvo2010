<div align="center">

<img src="./ascii.svg" width="460" alt="Nyvo"/>

<img src="./stats.svg" width="620" alt="Contributions in the last year"/>

[nyvo.is-a.dev](https://nyvo.is-a.dev) &nbsp;·&nbsp;
[linkedin](https://www.linkedin.com/in/niek-vogelaar-271222392/) &nbsp;·&nbsp;
[email](mailto:niek@nyvo.is-a.dev)

</div>

<img src="./hd-about.svg" width="620" alt="about"/>

> High school student from the Netherlands building AI applications and designing for the web.

I build fast, test on real users, and kill what doesn't work.

<img src="./hd-stack.svg" width="620" alt="stack"/>

<samp>python &nbsp; typescript &nbsp; javascript &nbsp; react &nbsp; html5 &nbsp; css3 &nbsp; vscode &nbsp; git &nbsp; github &nbsp; opencode</samp>

<img src="./hd-projects.svg" width="620" alt="projects"/>

**[Portfolio](https://github.com/Nyvo2010/Portfolio)** &nbsp;·&nbsp; <samp>typescript, react</samp><br>
My personal portfolio website showcasing my projects and skills.

**[Butterfly-Agent](https://github.com/Nyvo2010/Butterfly-Agent)** &nbsp;·&nbsp; <samp>typescript</samp><br>
Autonomous AI agent for various tasks.

**[Alto-AI-Backend](https://github.com/Nyvo2010/Alto-AI-Backend)** &nbsp;·&nbsp; <samp>python</samp><br>
AI backend service for intelligent systems.

<img src="./hd-stats.svg" width="620" alt="stats"/>

<div align="center">

<img src="./streak.svg" width="620" alt="Current and longest streak"/>

<img src="./langs.svg" width="620" alt="Top languages by bytes and by repo"/>

<img src="./year.svg" width="620" alt="The last year, one character per day"/>

</div>

<img src="./hd-about-this-page.svg" width="620" alt="about this page"/>

Every graphic here is generated inside this repository by a scheduled action.
The whole page makes **zero** third-party requests — no external services that
can rate-limit or go dark.

`ascii.svg` is a photo pushed through a character ramp (run `scripts/make_portrait.py`
once to create yours); the stat graphics and these section headings are drawn by
a [scheduled action](.github/workflows/stats.yml) straight from the GitHub GraphQL
API, once a day, committing only what changed.

They animate with SMIL inside the SVG, because GitHub strips `<script>` from
READMEs. The headings are SVGs for the same reason: GitHub also strips CSS, so
an image is the only way to put this page's own typeface on them.

The typeface is [JetBrains Mono](scripts/fonts), subset to just the characters
each graphic draws and inlined as base64. Language totals cover public
repositories only. `year.svg` uses the portrait's character ramp: ` ` `:` `+` `#` `@`,
quiet to loud.