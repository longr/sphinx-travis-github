# Automatically Build and Deploy Sphinx Docs to Github.io.

This is an example repository of how to build and deploy Sphinx docs to Github using Travis CI.  The files here will build the documentation and automatically deploy it to longr.github.io/sphinx-travis-github.

This tutorial will guide you through creating a blank repository, creating some documentation and deploying it to Github using Travis CI.  If you have an existing repository with sphinx documenation in it, you should be able to adapt these instructions to building your documentation automatically and deploying to github pages.

## Creating a repo

First we need to create a public repository on Github and clone it to your computer:

      git clone git@github.com:longr/sphinx-travis-github.git
      cd sphinx-travis-github

## Setup sphinx


If you have not installed sphinx already it is best to install it:

   pip install sphinx

if you are not in a virtual enviroment you may need to use the `--user flag`:

   pip install --user sphinx
   
Inside the main directory (sphinx-travis-github), create a directory called `docs` and run the sphinx quickstart in it.  You will be presented with several questions, you can accept the defaults or change them as you see fit.

       mkdir docs
       cd docs
       sphinx-quickstart

This should give you a layout like this:

     .
     ├── docs
     │   ├── _build
     │   ├── conf.py
     │   ├── index.rst
     │   ├── Makefile
     │   ├── _static
     │   └── _templates
     └── README.md

## Test the build

Before we attempt to automate the build we should check it works. From the `docs` directory and run the make command to build the html output.

       make html

This should build the html files in the `_build` directory.  We can check it built by opening the file `_build/html/index.html`.

## Automating the build.

The next section will not work until all three steps have been completed. Once all three have been completed

### Enable Travis CI on your repo

Before we can use Travis CI to build our documentation we have to link Travis CI and Github, and then enable Travis CI on our repo. Travis CI have great documentation on how to use it, https://docs.travis-ci.com/user/tutorial/. Essentially you need to:

1. Go to http://travis-ci.com and Sign up with you GitHub account.
2. Accept the Authorization of Travis CI. You’ll be redirected to GitHub.
3. Click on your profile picture in the top right of your Travis Dashboard, click Settings and then the green Activate button, and select the repositories you want to use with Travis CI.


### Creating a travis.yml file

Travis is controlled by a `.travis.yml` file located in the root of your repository. This tells travis how to build your documentation and where to deploy it to. Lets create a `.travis.yml` file that contains the following:

       language: python
       
       install:
        - pip install -r sphinx sphinx-rtd-theme

       script:
         # travis will start in the root of your repository so we need to
	 # change into the docs directory
	 - cd docs
         # Use Sphinx to make the html docs
	 - make html
	 # Tell GitHub not to use jekyll to compile the docs
	 - touch _build/html/.nojekyll

       # Tell Travis CI to copy the documentation to the gh-pages branch of
       # your GitHub repository.
       deploy:
         provider: pages
	 skip_cleanup: true
  	 github_token: $GITHUB_TOKEN  # Set in travis-ci.org dashboard, marked secure
  	 keep-history: true
  	 on:
    	   branch: master
  	 local_dir: docs/_build/html/

This file will first tell travis to use `python` and then install the python packages that we need `sphinx` and `sphinx-rtd-theme`. This second one is optional, but it is the theme I prefer to use so you will see it in the example files.

Next the script tells **travis** to move into the `docs` directory and run the command `make html`.By deafult github will parse files with jekyll to create a webpage, sphinx will build our webpage so we don't want jekyll to parse our files. To stop it from parsing them we then have travis create a file inside `_build/html` called `.nojekyll`.

The last part of this file **deploy**, copies all the files into an empty branch in our repository called `gh-pages`. Anything in a branch called `gh-pages` is automatically served onto github pages.  Here we tell travis which files to copy to the repo, in our cases, everything in `docs/_build/html`.

### Create a personal access token

One of the last parts of the travis script mentions a **github-token**; this token is need so that travis has permission to write to our repository.  You can find out how to get and use this token by reading about GitHub Pages deployment: https://docs.travis-ci.com/user/deployment/pages/#Setting-the-GitHub-token.

## Triggering the build

Everything should now be setup so that everytime we push changes to GitHub our documentation will be build and deployed to longr.github.io/sphinx-travis-github.

First we will need to add our docs directory to our git repository. We will want to exclude the `_build` directory 


## Adding the files to git

Our documentation will be built when we push our files to github, we cannot push them to github until we have added the files and committed them.  We will want to add all of our source files, but exclude our build directory.  We can exclude the build directory by creating a file called `.gitignore` this file needs to contain the files or directories we want to exclude so it should look like this:

    _build

We can now add the remaining files.  From the project root directory (sphinx-travis-github) stage and commit the files, and then push them to GitHub with the following commands:

   git add .
   git commit -m 'First commit of docs'
   git push

This should send our files to github, travis will then build and deploy them and the results will be hosted on username.github.io/repository-name, in this case: longr.github.io/sphinx-travis-github
