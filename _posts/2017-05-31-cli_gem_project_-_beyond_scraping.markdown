---
layout: post
title:  CLI Gem project - Beyond Scraping
date:   2017-05-31 19:32:45 +0000
---


Because I’m a photography hobbyist, I have a fetish to stare at the priciest gears there are to buy every so often. That is why I chose to scrape B and H Photo Video’s digital camera section sorted by bestsellers category. I decided to go for the extra bonus points by creating and publishing my gem in Rubygems.org. Along the way, I learned other stuffs about coding than just scraping. 

I used Bundler to create and publish my gem. After quickly bundling the gem and tweaking the bin file, I did a test run. Not so fast. I had to first edit the gemspec file: type in all the TODOs, edit the bindir to *‘bin’* from the default *‘exe’*, comment out the unnecessary scripts, and add some dependencies. Finally, “Hello World!!!”

I had the design pattern of my program similar to that of Student Scraper lab with the Scraper methods spitting out hashes and my Camera objects instantiated with a hash. I had to refactor it after speaking with the instructor. More on that later. I used a rake console task to test all my scraping codes. During the coding, I also got to learn some new tricks with git *rebasing* when I had to correct the comment I had made a few commits earlier. I also had to *force push* to git repository because I started the repository with a README file. Finally the gem was built and published.

For the refactoring, I decided to change the major version of the gem. I tweaked the Scraper, Camera and the CLI class to now accept attributes instead of the hash. The pre-refactoring version would load all the camera objects from the list page and the attributes of all the cameras from the detail page at the start of the program. The refactoring also made the program much faster by only loading the list page at the start of the program and loading the attributes from the detail page only when requested by the user. 

