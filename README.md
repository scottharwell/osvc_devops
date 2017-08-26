Service Cloud Development Hooks
===============================
This repository is a collection of git hooks that can help introduce a dev/ops workflow when building Oracle Service Cloud customizations.  The hooks expect a few organizational requirements in order to work, but they can be customized to meet your workflow.

For the purposes of this doc, assume that we're working with three branches in Git, `master` for production, `develop` for our testing and feature merges, and then `test` as our feature branch.

## Environment Setup

These hooks will work on any OS, provided that you have PHP installed and in your PATH with the cURL module enabled in php.ini.

## Repository Setup

These hooks expect a certain organization to your OSvC development Git repository.  The Customer Portal folder, in particular, must follow the structure below.  Notice that after the Customer Folder parent, the folder structure matches an interface's WebDAV structure.

* .git
* .gitignore
* Add-ins
* CPMs
* Custom Scripts
* Customer Portal
    * Interface_1 (i.e. "sharwell" or "sharwell__tst")
        * dav
            * cp
                * customer
                    * assets
                    * development
                        * (dev folders)
    * Interface_2
* (other folders...)

The important part is the Customer Portal folder.  The hook expects this folder and that its contents are broken out to match the /dav/cp/... structure that applies to all OSvC sites.

## Git Configuration

The hook relies on a few configurations to connect to OSvC.  Those properties are:

### Required Configs

* `osvc.webdav.url "https://sharwell.myrightnowsite.com"` (URL to WebDAV root)
* `osvc.webdav.username "webdav_username"`
* `osvc.webdav.password "webdav_password"` (**Note:** This does store your webdav password in plain text in your git configs.  Be sure to treat this with care.)
* `osvc.webdav.branch test` (The working branch that will be used to determine if pushes to webdav should happen.  Any commit not on this branch will not push to WebDAV.)

#### Optional Configs

* `osvc.webdav.proxy "http://proxy.url"` (used only if your network requires an HTTP proxy, which is required for cURL to operate in some of these hooks.)

The primary caveate in this workflow is that you can only work on one interface at a time since the `url` parameter limits pushing to a particular interface.

# Hook Usage

The following section outlines how to configure your Git repo and these hooks in order to work with your OSvC environment.

We asume that you are managing OSvC development in a one-repository-per-site format; meaning, all customizations for one site (including multiple interfaces) are in the repo.

## Hooks

Each hook in this repo would be placed in your `.git/hooks` folder for each repo that you want to apply them to.

### post-commit

The `post-commit` hook runs after each commit to the git repo.  The purpose is so that dev work can be done and tracked through git on a local machine, and then the hook publishes the changes made to WebDAV.

My typical workflow with this hook is to have an active working branch called `test`, which is the branch I use in my `osvc.webdav.branch` config (see below).  I then have a console open and run `git commit -am "test"` for each save.  This captures every save that I make in a history, and I can make the commit message more applicable, but the log usually gets really long.  Once I have my tests done, I then merge the `test` branch into develop with a squash merge.  Below outlines the process in steps.

1. Checkout `test` branch from `develop` (`git checkout -b test`)
2. Make code changes to one or more files.
3. Run `git commit -am "test"`
    * The post-commit hook will run and push my changs to the active site/interfance automatically.
4. Test my changes.  Repeat steps 2 and three as needed.
5. Checkout develop branch (`git checkout develop`)
6. Squash merge my test branch (`git merge -Xtheirs --squash develop`)
7. Delete my test branch (`git branch -d test`)

![animated gif](https://github.com/scottharwell/osvc_devops/blob/master/img/post-commit.gif?raw=true)