* [Readme Guideline](#readme-guideline)
   * [What the purpose of ReadMe?](#what-the-purpose-of-readme)
   * [How to achieve this purpose?](#how-to-achieve-this-purpose)
   * [Reference](#reference)
* [Digression: create TOC](#digression-create-toc)

# Readme Guideline



## What the purpose of ReadMe?

The purpose of readme in a project is to make new developers start work quickly and make a project more maintainable.



## How to achieve this purpose?

1. Several words to introduce this project
2. List professional glossaries about this project
3. Share some information to help developers build and run this project
4. List the project modules  that helps developers know more about the structure of this project 
5. List the technology stack that helps developers know what tech they should learn
6. List the team roles of the project
7. Link a development workflow guideline or simply introduce the development workflow of this project
8. Link a code style or introduce a code style that developers must follow
9. List some specific technical detail that must be unified
10. List some terms and abbreviations that may stick new developers
11. Link a API doc if it is a front-end or app project
12. Link a design and REQ doc
13. Link or introduce steps to help developers release project
14. A big and nice project sometimes have a wiki or blog for developers to record valued researches, solutions and tedious information



**The points above are not necessarily all included in readme, but more is better**



## Reference

Template: [Template Sample](ReadMe%20Template.md)

Sample: [Japanese Macdonald's App ReadMe](Japanese%20Macdonald's%20App%20ReadMe.md)



# Digression: create TOC

Github is not support TOC syntax of markdown. So here is a solution:

Use [gh-md-toc](https://github.com/ekalinin/github-markdown-toc) to print TOC and copy-paste to the top of markdown file

Usage: 

1. Terminal: ./gh-md-toc Readme\ Guideline.md 
2. Copy-paste the print content to the top of markdown file

