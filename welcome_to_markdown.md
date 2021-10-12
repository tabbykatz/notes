# Hey there, Markdown-curious folx

![markdown logo](https://upload.wikimedia.org/wikipedia/commons/thumb/4/48/Markdown-mark.svg/175px-Markdown-mark.svg.png)

<p>This will be a hastily assembled rundown of how I suggest you get started in the wonderful world of Markdown + how to create a Github profile README.</p>

> fun fact: [I fell in love with Markdown when I had to write a simple python Markdown interpreter](https://github.com/tabbykatz/Markdown2HTML)

## Why

Markdown is a very powerful, portable, and prolific markup language. You've definitely used it in one way or another, and learning to be more effective with it is a skill you should develop.

Markdown can be used to create websites, documents, notes, books, presentations (I almost made today's presentation with [remark](https://remarkjs.com/#1) or [marp](https://marp.app/) but didn't have time), email messages, and technical documentation.

Markdown was designed with the explicit intention of being easily readable by humans.

## Start with Gruber

He [Literally Wrote the Language](https://daringfireball.net/projects/markdown/)

## Let's assume you're using VSCode

[Learn VSCode + Markdown specific stuff](https://code.visualstudio.com/docs/languages/markdown)

Use Prettier to format Markdown. Use Prettier for everything, all the time. Format on save.

Have questions about Prettier and setting up VSCode in general? Maybe we should have one of these just for that... but back to Markdown:

Some examples to play with:

```css
.h1 {
  color: red;
}
```

```python
def something():
  print("something");
```

| This | is   | a      | table |
| ---- | ---- | ------ | ----- |
| I    | love | Tables | !     |

_hey_ **hey**... **_hey_**

- [x] here's a task list
- [x] list syntax required
- [x] this is a complete item
- [ ] this is an incomplete item

You can use html markup, too, because Markdown all becomes html in the end.

html: <strike>strikethrough</strike><br> markdown: ~~strikethrough~~

<details>
<summary>title of the content</summary>

stuff in here

</details>

[But read a bit before you try to mix these too much.](https://daringfireball.net/projects/markdown/syntax#html)

<style type="text/css" rel="stylesheet">
 * { color: ; }
</style>You can add CSS too!

### LaTeX

<img src="https://render.githubusercontent.com/render/math?math=e^{i \pi} = -1">

![formula](<https://render.githubusercontent.com/render/math?math=\large\f(x)=sin(x)>)

VS Code supports Markdown files out of the box. You just start writing Markdow text, save the file with the .md extension and then you can toggle the visualization of the editor between the code and the preview of the Markdown file; obviously, you can also open an existing Markdown file and start working with it. To switch between views, press ⇧⌘V in the editor. You can view the preview side-by-side (⌘K V) with the file you are editing and see changes reflected in real-time as you edit.

## Or use Marked/similar to view live previews in different flavors of Markdown

- [marked](https://marked2app.com/)

There are many different Markdown flavors. Today we care about Github flavored Markdown because we want to use this awesome markup language to make a profile README on GitHub.

- [Github's Mastering Markdown Guide](https://guides.github.com/features/mastering-markdown/)

## How to make the README

Create a repository with your username as its name. I'm @tabbykatz, so:

[https://github.com/tabbykatz/tabbykatz](https://github.com/tabbykatz/tabbykatz)

A file named `README.md` in the root folder of this repo will show up on your profile page as a profile Readme.

You can make it as simple or as complex as you like. Mine is very simple. Here is the raw Markdown to produce mine:

```markdown
### Hi there <img src="https://raw.githubusercontent.com/tabbykatz/tabbykatz/master/wave.gif" width="30px">

[![Typing SVG](https://readme-typing-svg.herokuapp.com/?lines=I'm+Tabitha.+Get+Vaccinated!)](https://git.io/typing-svg)

[![Twitter Badge](https://img.shields.io/badge/-@tabby__katz-00acee?style=flat&logo=Twitter&logoColor=white)](https://twitter.com/intent/follow?screen_name=tabby__katz "Follow on Twitter")
[![Linkedin Badge](https://img.shields.io/badge/-tabbykatz-blue?style=flat-square&logo=Linkedin&logoColor=white&link=https://www.linkedin.com/in/tabithaomelay/)](https://www.linkedin.com/in/tabithaomelay/)
[![Instagram Badge](https://img.shields.io/badge/-tabby_katz-purple?style=flat-square&logo=instagram&logoColor=white&link=https://instagram.com/tabby_katz/)](https://instagram.com/tabby_katz)
[![Medium Badge](https://img.shields.io/badge/-@tabbykatz-03a57a?style=flat-square&labelColor=000000&logo=Medium&link=https://medium.com/@aemmadi/)](https://medium.com/@tabbykatz)
[![Gmail Badge](https://img.shields.io/badge/-tomelay@gmail.com-c14438?style=flat-square&logo=Gmail&logoColor=white&link=mailto:tomelay@gmail.com)](mailto:tomelay@gmail.com)
![Github Stats](https://github-readme-stats.vercel.app/api?username=tabbykatz&count_private=true&show_icons=true&include_all_commits=true)
![Top Langs](https://github-readme-stats.vercel.app/api/top-langs/?username=tabbykatz&hide=TeX&layout=compact)
```

- [Sarah Graywood had fun making hers](https://github.com/smgraywood)

- [Here is a curated list of awesome profile readmes](https://github.com/abhisheknaiidu/awesome-github-profile-readme).
  View some of the code and see what you can do with it!
- You'll notice that you don't have to just use Markdown.
- [You can even automate the process](https://www.freecodecamp.org/news/go-automate-your-github-profile-readme/)
- [You want it to look like MySpace, you say?](https://dev.to/bdougieyo/making-myspace-in-2020-5glj)
- [are you curious about syntax highlighting in Markdown codeblocks?](https://support.codebasehq.com/articles/tips-tricks/syntax-highlighting-in-markdown)
- [listen, Markdown is really cool and I could keep listing links all day](https://gist.github.com/DrSensor/b2a674f42933b2ad6db4c6f7934b32a5)

I will probably never tire of learning about Markdown and the nifty things humans do with it. If you feel similarly or need help in the future please reach out, I'll be happy to explore with you.
