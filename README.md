[Strikingly](https://www.strikingly.com) tech blog powered by Hexo. Thoughts and learnings from buliding Strikingly.com. Please visit: [http://strikingly.github.io/blog](http://strikingly.github.io/blog)

Supported categories:

* Frontend
* Backend
* iOS
* DevOps
* Design
* UI/UX
* English
* Team
* 中文

## Getting started
* Clone the repo
* cd to the directory
* Install the dependencies

  ```npm install```

* Install hexo to your local machine

  ```npm install -g hexo-cli```

* Install the 'next' theme for hexo

  ```git clone git@github.com:iissnan/hexo-theme-next.git themes/next/```

* Start server

	```hexo server```

## To publish a post

* Fork this repo
* Use collaboration tools like Quip or Google Doc to write the article and get feedback from team members. (We recommend Quip since it supports markdown export)
* Use hexo to create a new post

```hexo new <title>```

It will generate a post md file and a folder for you to put static assets.

* Write/copy the markdown to the md file
* File a PR
* Once the PR is merged, run this command to publish it online

```hexo generate --deploy```

This will generaete the static files from the local git repo and publish it to Github pages.


## Getting ideas

Sometimes you are not sure if people will be interested in a topic you have in mind. Feel free to file an issue and
bounce ideas with the team. There are also topics in issues list that people want to learn more about but nobody claimed it yet.

If you ran out of idea, find an English tech article that you really like and translate it to Chinese.


**Every dev who passed probation will be writing a blog post every two months, people who failed to do so will buy a round of :beers: or :pizza: for all devs**
