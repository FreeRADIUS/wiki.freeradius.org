# Submitting patches via GitHub
## Introduction
GitHub is a social coding site allowing many developers to come together and work on projects simultaneously.

GitHub is based on the Git version control system. What makes Git different from other version control systems (like CVS and SVN), is that there is no need for a central master repository. When you _clone_ (checkout) a repository, you get your own private repository containing all the commits and branches of the original. 

After cloning a repository you can create new branches and commit new revisions completely independently of the _origin_ repository. To distribute and receive changes you can use the _git pull_ (to get new commits from remote repositories), and _git push_ (to send your commits to remote repositories).

## Installing Git
See [here](http://book.git-scm.com/2_installing_git.html)

## Getting a GitHub account
In order to follow or fork a project you must first create a GitHub account. The current signup url is [here](https://github.com/signup/free) and the process is pretty straight forward. After you've signed up, log in and you're good to go.

Another advantage of having a GitHub account is its one of the providers this wiki supports for login, so even if you never contribute to the source code of the server, you'll still be able to help out be adding wiki content. 

## Authorising your local machine to talk to GitHub
In order to let your local machine talk to GitHub, you must first lodge the public portion of a RSA keypair with the GitHub servers.

### What is a RSA keypair?
See [here](http://en.wikipedia.org/wiki/Public-key_cryptography) and [here](http://en.wikipedia.org/wiki/RSA)
### How do I generate a keypair?
If you're on a Mac or Linux use the following command
    ssh-keygen -t rsa
And save the keys if their default locations (just hit return)

@todo - windows instructions - Currently your only option is to cry yourself to sleep knowing you'll never be able to contribute to FreeRADIUS (or figure it out on your own).
### How do I install the public key on GitHub's servers?
* Use the following to output your public key block to the terminal
      cat ~/.ssh/id_rsa.pub
* Copy the key (making sure you don't accidentally get bits of command prompts).
* On the GitHub site go to [Account Settings->SSH Public Keys](https://github.com/account/ssh), then click the _Add another public key_ link.
* Enter a Title for the key (this is arbitrary, just enter your machine's hostname or any other value you want).
* Paste your public key into the Key field
* Click _Add key_

## Linking your commits with your GitHub account
Once you've established a history of providing good patches, the FreeRADIUS core developers are more likely to pull your modifications into the project repository. To establish this history it's important to configure your local git client to insert the correct information into commit meta-data. This will mean that your commits are linked to you and your GitHub account.

To correct the default commit user.name and user.email, enter the following commands
    git config --global user.name "<Your real name>"
    git config --global user.email "<Email you used to signup with GitHub>"

## Forking
In order to contribute code you must first fork the server. Forking is the process of creating your own copy of the FreeRADIUS repository on GitHub, which you can then clone and _push_ your modifications to.

To fork the FreeRADIUS server, login to GitHub, then go to the [project site](https://github.com/alandekok/freeradius-server) and click the 'Fork' button. You'll see the current URL change to ``/<your github username>/freeradius-server`` this means you're now browsing your own copy of the FreeRADIUS source code.

From this point on changes to the original repository will not be reflected in your fork unless you explicitly sync up with the _origin_ repository.

## Using your shiny new fork
GitHub doesn't allow you to login to their servers with an interactive prompt. Instead you must use one of the protocols Git supports (SSH, HTTP, or Git (native protocol)) to modify the contents of your fork, and in order to do that you must first _clone_ it (create a copy of it on your local machine).

### Cloning
* To clone your fork create a directory to house your repositories and navigate to it
      mkdir ~/repositories/
      cd ~/repositories
* Use git to clone your fork
      git git@github.com:<your github username>/freeradius-server.git
      cd freeradius-server
* Add the original FreeRADIUS repository as a remote branch (you need this to pull new commits from the project repository)
      git remote add upstream git://github.com/alandekok/freeradius-server.git
      git fetch upstream
    
### Making modifications
Before you start hacking away, you must create a new branch. This branch will serve to group your commits together so that they can be merged back to the project repository later.

#### Branching 
* Think of a descriptive name for your branch, something like 'rlm_foo_segfault_issues'
* Navigate to your local git repo
      cd ~/repositories/freeradius-server
* Create the branch locally
      git branch <my new branch>
* Push the new branch up to your fork
      git push origin <my new branch>
* Switch to your new branch
      git checkout <my new branch>
      
#### Commits and pushing changes
* Hack away at the server code or apply a pre-existing patch or set of diffs
* Tell Git which files you want to include in the commit using ``git add <path>``. If you add subdirectories, all files and directories in that subdirectory will also be added.
* Commit your changes
      git commit --message '<description of changes>'
* Sync your branch up with upstream to make sure there are no conflicts
      git pull --rebase upstream master 
* Push your changes back to your fork
      git push
      
#### Rebasing
@todo

### Generating a pull request
The reason why the FreeRADIUS core developers love GitHub is because of pull requests. Once you've committed a set of modifications to your fork, you can generate a _pull request_ to let one of the developers know you have code to merge.

If your code merges cleanly (which it should as you just pulled down the upstream commits, right?), then its a couple of clicks to get it merged into the main repository. This is **significantly** easier for core developers, compared with attempting to apply mangled patches sent on the mailing lists.

Pull requests are generated from within GitHub with a commit hash.

* Navigate to your local git repo
      cd ~/repositories/freeradius-server
* Use the following command to get the hash of the last commit on the current branch, then copy it
      git log -n 1 --pretty=format:%H
* Go to your fork on GitHub _https://github.com/your github username/freeradius-server_
* Click the 'Pull request' button 
* Follow the instructions...