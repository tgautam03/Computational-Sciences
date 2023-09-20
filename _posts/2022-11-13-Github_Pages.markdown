---
layout: post
comments: false
title:  "Github pages for the repository"
excerpt: "Setting up individual website for the repository"
date:   2022-11-13 09:00:00
---

### Prerequisites
I'm assuming that:
- Jekyll is installed and you know how to work with it (it's relatively easy and you can check the links provided at the end of this post).
- Main Github Pages website is setup and ready ([here's mine](https://github.com/tgautam03/tgautam03.github.io)).
- You have your website layout ready (I forked Andrej Karpathy's [blog website](https://karpathy.github.io/) from his [github repository](https://github.com/karpathy/karpathy.github.io)).

### Setup
- The very first thing I do is create */docs* folder in the main repository (example [here](https://github.com/tgautam03/Algorithmic-Toolbox)). 

- After this I just used the configuration files from Karpathy's repository (obviously I didn't plagarise his work and only used the website layout which is open-source) and ran `bundle install` command to update website in accordance with *_config.yml* contents.

	> I ran into an error involving *charlock_holmes* and resolved that by `sudo apt install libicu-dev`. Also the website didn't deploy until I installed `bundle add webrick`. 

	> If you want to you build your own website layout, use the command `jekyll new --skip-bundle .`

- Now, commit and push the changes to remote github repository.

- Next, go to *Settings/Pages* on Github and select the `main` branch alongside the `/docs` folder under *Build and deployment* section.

- Finally, copy the markdown file to main github pages's posts folder too (for some reason it only pick links from main website).

### Conclusion
This post covers:
- Setting up repository website using Github Pages.

### Useful Links
- [Github Docs](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/creating-a-github-pages-site-with-jekyll)
- [Jekyll](https://jekyllrb.com/)
- [charlock_holmes error](https://github.com/github/linguist/issues/3878) 
