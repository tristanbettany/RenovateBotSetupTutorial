# Renovate Bot Setup Tutorial

How to get started with Renovate Bot

## Preamble

This tutorial should give you a good start in getting setup with Renovate Bot on your version control platform of choice.

Renovate is massivley configurable so I wouldnt suggest using the final setup of this in a production environment without first
checking to see if theres other configuration options that can help control your workflow better, it is purley designed to
get you up and running and from there you can continue configuring.

An assumption I am making during this tutorial is that you will be setting renovate up on a repo that has dependancies managed
using npm, commonly a node application. This means by installing node to run renovate it will have what it needs to check the dependencies.

If you are running this against a repo with java, php, or other dependencies then you will need to make sure that the machine running 
the renovate node application is also running the dependacy management tools for those things such that renovate can work correctly.
For example composer would need to be installed for PHP.

## Authentication setup

Considering every team I have setup renovate in before has been using Bitbucket I will move forwards with this tutorial on how 
to set up renovate to communicate with bitbucket cloud.

You will need a personal access token for github and an app password for bitbucket. The reason for also needing one for github 
is due to alot of dependancies renovate needs to deal with will be on github.

If working in a production environment it is ideal to set up a seperate account on the version control platforms just for renovate 
and generate the app password and token on these seperate accounts. For this tutorial though and setting up a POC you can use your own accounts 
to generate these.

- https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens
- https://support.atlassian.com/bitbucket-cloud/docs/create-an-app-password/

Once you have gathered your authentication tokens / username and password, note them down and save them for later.

## Configuring your repositories

Each repo that you would like renovate to manage needs a configuration file on the master branch to tell renovate how to operate.
This config file cannot live on a seperate branch otherwise renovate will not work or will try to create a PR with boilerplate 
config to that repo. You should ALWAYS have the configuration file on the master branch in the root of the repo and up to date with your expectations.

So... at the root of your repo create a file called `renovate.json` and put the following code in it.

```json
{
	"extends": [ "config:base" ],
	"branchConcurrentLimit": 50,
	"prConcurrentLimit": 50,
	"prHourlyLimit": 50,
	"patch": { "enabled": true },
	"minor": { "enabled": true },
	"major": { "enabled": true },
	"rebaseWhen": "never"
}
```

`branchConcurrentLimit`, `prConcurrentLimit`, and `prHourlyLimit` have been set to 50 here meaning that 50 PRs can be created by renovate at the same time
and a max of 50 PRs per hour in this repo. You can change this and I recommend you lower it for significantly small teams or individuals otherwise it could
give you alot of stuff to sort out in your repo.

`patch`, `minor`, and `major` should allow you to control how far renovate bumps dependancies. You can also use this config to control much more than what is just here,
but this should be plenty to get you started. To see more configuration options for this file, look here - https://docs.renovatebot.com/configuration-options/

Once you have completed your config file make sure to commit it to master!

## Configuring the Renovate Bot application

First off you need to install nodeJs, you can install this from package managers on linux or using an installer on windows.

Once installed you can create a folder for your renovate bot configuration and binaries.

Iniside that folder create a package.json file and put this code in it.

```json
{
  "dependencies": {
    "renovate": "^37.128.0"
  }
}
```

Now type `npm install` which will install renovate and save its binaries in this folder.

You now need to create a file in the root of this folder called `renovate.js` and put the following code in it

```js
module.exports = {
  platform: 'bitbucket',
  username: 'mycoolusername',
  autodiscover: true,
  repositories: ['myprojectname/myreponame']
};
```

Replace `mycoolusername` with the username of your bitbucket cloud account, this is not your email address nor your display name. 
A username is all lowercase with no spaces.

Replace `myprojectname/myreponame` with the path to your repo on bitbucket cloud. You can add more repos here given this is an array.

There is a tonne more configuration for the self hosted node version of rennovate which can be found here - https://docs.renovatebot.com/self-hosted-configuration/

I would encourage anyone using this seriously to go through the options there and see if theres anything that would be useful for them to add 
to the config. 

Now youve installed and configured the renovate node application you need to run it alongside a little more configuration.

The easiest way I found to do this is to create a bash file, other people may prefer alternative methods, but this met my requirements.

Create a file called `exec-renovate.sh` in this folder your in alongside your configuration and put the following code in it.

```bash
#!/usr/bin/env bash

export LOG_LEVEL=debug
export RENOVATE_CONFIG_FILE="./renovate.js"
export RENOVATE_PASSWORD="my-bitbucket-cloud-app-password"
export GITHUB_COM_TOKEN="my-github-person-access-token"

node ./node_modules/renovate/dist/renovate.js
```

If you dont want to see all the output of renovate then remove the env var `LOG_LEVEL=debug`, however i keep it in as it makes
it easy to see issues when renovate isnt playing nice.

Replace the value in the env var `RENOVATE_PASSWORD` with the app password you got from creating one in bitbucket cloud.

Replace the value in the env var `GITHUB_COM_TOKEN` with the personal access token your generated in github.

Now make the bash script executable by running `chmod +x ./exec-renovate.sh`

NOW RUN IT! :)

You should have PRs been made by renovate assuming that it found dependancies that need updating.