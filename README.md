# Setting-Up-An-Automated-Hugo-Site
A guide to setting up a Hugo site on GitHub that automatically rebuilds when changes are made


# Setting up an Auto-Deploy Hugo Site
The idea behind this method of setting up a Hugo site is that by using GitHub pages and GitHub actions, you can set up a full Hugo ecosystem on GitHub that will automatically build and deploy every time a change is made to the site.
This is done by having 2 GitHub repositories.  One for the front end that just hosts the GitHub Pages content, and one for the backend which houses the Hugo builder and theme files.  The backend site also has a GitHub action in place that watches for commits and then rebuilds the site and pushes the new version to the front end whenever a change is made.

# Prep
Things you need to do before you start:
- Install Hugo on your computer and make sure it is added to your PATH (find the instructions for your OS here)
- Make sure you have a GitHub account (get one here if you don’t)
- Install git on your computer (Instructions found here)
- Choose the Hugo theme that you will want to use (find a list of themes here)
- Create a repository for the front end of your site.  You don’t need to add a README file or anything else to this site.  Give it a clean name as this will be in the URL of your new site (unless you are making a custom domain)
- Create a second repository for the backend of your site.  Make sure not to add anything to this site yet, not even a README file unless you are comfortable using git from the command line to resolve conflicts.  This repository can be named whatever you want but a good rule of thumb is to name it the same as your front end site but with the word SOURCE or BACKEND in front. (eg. “SOURCE-my-website-name”)

# Building the Site
The first step is to open a command line interface on your machine, navigate to a folder that has access to both Hugo and Git, and to make build your base Hugo site using the command below.
'code' hugo new site NameOfSite 'code'
Now that you have a Hugo site prepped, move into the site folder using the cd command (cd NameOfSite), initialize the folder as a git repository (git init), and add your theme.  In most cases, this will be done by adding the theme as a git submodule and the code to do this should be listed on the webpage for the theme.  An example of what that code should look like can be seen below (the examples shows adding the “Blist” theme).
git submodule add https://github.com/apvarun/blist-hugo-theme.git themes/blist
Once the theme is installed as a submodule, look at the theme’s documentation (found on the Hugo themes directory) to see if it requires configuration.  Very basic themes might work simply by adding it as a submodule but most of the more complex themes require additional steps.  In the case of the Blist theme used in this example, the documentation walks you through how to set up some Node JS features and a css manager called PostCSS that are needed to use the theme.
And just like that you have a Hugo site with a theme!  Now we need to adjust the config.toml file to reflect the correct url for your site.  Open up the config.toml file in the root folder of your site and update the baseurl field to reflect your sites location.  Use the format below by replacing GITHUBUSERNAME with your username or the name of the GitHub organization that the site is in and replace NAMEOFSITE with the name of your front-end site.
baseurl = "https://GITHUBUSERNAME.github.io/NAMEOFSITE/"
Now that you have your base url set up, its time to have Hugo build your site!  This is done by simply running the hugo command.  Hugo will build your site and output it into a new folder called “public”.  The public folder is what we are going to deploy on our front-end site.

Setting up the front-end of your site
To upload the contents of the public folder to GitHub, use the cd command to go into the public folder (cd public), initialize the folder as a git repository (git init), and then use the normal git upload procedures as shown below (replacing the capitalized placeholders with you sites info).
git remote add origin https://github.com/GITHUBUSERNAME/NAMEOFSITE.git
git add .
git commit -m "Initial commit"
git push --set-upstream origin master
With that your site should be uploaded to GitHub.  Next you need to make it live on GitHub Pages by going to your front-end site on GitHub, clicking on Settings, clicking “Pages” in the left menu, then adjusting the dropdown under the “Source” header to “master” and clicking the save button.
After a minute or two your front-end site will be live at the link shown on the screen.  At this point your front-end site is ready to go and you should never need to interact with this repository again!

Setting up the Back-end of your site
Back on the command line, navigate back to the root of your site (if you are in the public folder you can do this using the cd .. command).  
Before we move the Hugo builder up to our back-end repository, we need to remove the public folder containing our site.  The reason we want to do this is because we are hosting the output of the site builder in a different repository.  You can do this from the command line using the appropriate delete command for the operating system you are using or you can use the file explorer to delete the public folder.
Once the public folder is removed, the steps to move your builder to GitHub are the same as when you set up the front-end site except you are uploading to your back-end repository.  
git remote add origin https://github.com/GITHUBUSERNAME/NAMEOFBACKENDSITE.git
git add .  (This might look like a lot depending on what theme you are using)
git commit -m "Initial commit"
git push --set-upstream origin master
With that your Hugo builder is now fully online!  

Setting up Permission and Secret
Now that our back-end site has all the pieces needed to run Hugo, we need to set up an action in GitHub that will re-build and re-deploy our site to the front-end whenever we make a change.  In order for a GitHub Action to push and pull to other repositories we need to give that action permissions through what’s called a Personal Access Token.  If you do not already have a personal access token on GitHub, you can make one by following the steps below:
•	Go to your User Setting in GitHub (click the avatar in the top right and select settings)
•	Navigate to Developer Settings (found at the bottom of the menu on the left)
•	Open your personal access token list (from left menu)
•	Click “Generate new token”
•	Fill out the form
o	It is recommended to describe what the token is going to be used for in the note section
o	For expiration it is easiest to set no expiration date but be aware that there is a risk to doing this.  (read the documentation linked in the warning for more information)
o	If this token will only be used for hugo site automation as described in this guide then you only need the “repo” and “workflow” scope checked
o	Click “Generate token” at the bottom of the page
•	Your token will now be displayed in the token list
•	COPY YOUR TOKEN TO A SECURE LOCATION as this is the only time you will ever be able to see your token.  If you lose it you will need to make a new one.
Once you have a personal access token, go back to your back-end repository and open the settings.  From the left hand menu, click “Secrets” and the “Actions”.  This is where you can create encrypted variable for use in your GitHub actions.  Keep in mind that anyone with collaborator or higher access to your repository can use these variables.
Create a new secret by clicking the “New repository secret” button.  Here you will give a name to the variable that will be used in your actions and then paste your personal access key into the “Value” box ensuring that you do not add any extra characters or whitespace.  You can name your variable whatever you like but for the purposes of this guide we will call it PERSONAL_TOKEN as that is what it contains.  Once you have named the variable and pasted your token, click “Add secret” to create it.

Building the Action
Now that we have our permissions variable, it is time to write the action that will do handle our automation.  To begin, go to the Actions section of your back-end repository.  Since we are just making a single action, we will click the “set up a workflow yourself” option near the top of the page which will create a new action for us and open the editor.
Read through the template provided to get an idea of how to set up an action.  Below is an example action that works for the Blist theme with notes explaining each piece.  Notice the task that install dependencies.  Make sure whatever dependencies your theme needs are installed before the Buid task.

#Name of action
name: hugo CI

#This makes the action trigger on any push to the master branch of the repository
on:
  push:
    branches: [ master ]

#This is the list of tasks that the action will execute.  
#They will be executed in order an in an Ubuntu environment
jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      #This task checks out the git repository so it can be used by your action
      - uses: actions/checkout@v2
        with:
          submodules: true 
          fetch-depth: 1   
      #This task sets up hugo in the actions environment
      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: 'latest'
      #This task installs the dependencies required for the "Blist" theme
      - name: Install dependencies
        run: |
          npm install postcss -D
          npm install -g postcss-cli
          npm install -g autoprefixer
      #This task executes the hugo build command
      - name: Build
        run: hugo
      #This task pushes the build to the Front_End repository
      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          personal_token: ${{ secrets.PERSONAL_TOKEN }}
          external_repository: GITHUBUSERNAME/NAMEOFSITE
          publish_branch: master
          publish_dir: ./public


Once you have your action built, commit it and it will run automatically.

Wrap Up
And with that you are all set!  From now on when you make a change to your hugo site files in the back-end repository, it will automatically build a new version of your site containing those changes and push it to your front-end repository!
