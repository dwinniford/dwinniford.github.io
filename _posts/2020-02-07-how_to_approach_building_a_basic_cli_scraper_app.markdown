---
layout: post
title:      "How to approach building a basic CLI scraper app"
date:       2020-02-07 14:42:20 -0500
permalink:  how_to_approach_building_a_basic_cli_scraper_app
---

Starting a program from scratch can be intimidating.  Here are the steps I followed to help me get started (and keep going).
.
### Start with the user interface.

Don't worry about all the complex coding you will need to do later.  Start with what you want the user to see when the program starts.

### Build it one step at a time.

For example start with writing the code that will puts the main menu.

### Try it out after every addition and correct errors along the way.

Does the main menu work?  Add another layer of the menu and test again.

### Make it work with puts-ing fake data.

Don't worry about the complicated steps of getting the real data yet!  Write in some example data to output in the terminal as a place holder.

### Separate concerns in your CLI class.

Can you make your code more clear by separating code into new instance methods?
	
### What other classes will you need and how will they work together?

Write these classes into your CLI code.  Go ahead and write the names of the class and instance methods that you want to have and make notes about what arguments they will need and their return value.

### Pick one class to start coding.

Look at the instance methods that your CLI class needs to call.  Build those methods one at a time.  If applicable, move fake data from your CLI class to the new class you are creating.  Does your CLI work correctly with this class?  Test after each addition!  Remember you're still using fake data!
	
### Repeat with the next class!

### Replace fake data.

Once your classes are all working correctly with your CLI class start replacing your fake data with code to pull from an API or scrape from a website.  As always test your program after each addition.  If you're scraping, this might be the most tedious and time consuming part of your project.  

### Refactor

Now that you're using real data your program should be almost done!  Take a step back to reconsider your code.  Can you refactor anything?  Do you need a new class to separate concerns?

### A few more notes:  

Expect your code to break every time you test it after an addition.  This is how you find syntax errors and coding problems!  Its good to check on your code in small batches so it doesn't get overwhelming!  

**Don't forget to push your changes to Github frequently!**
