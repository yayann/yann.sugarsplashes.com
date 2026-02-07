---
title: "Automating Deployment for PHP and Javascript Apps Using a Chat Bot"
date: 2015-05-03
tags: ["grunt", "hubot", "rocketeer", "shipit"]
categories: ["automation"]
summary: "How I set up a chatbot-driven deployment workflow for PHP and JavaScript applications using Rocketeer, Grunt, Shipit, and Hubot."
showToc: true
---

I've set up a sweet deployment workflow at work: any developer on the team can order the bot in our company chatroom to deploy a web app (CMS, API server, or Angular apps) from the git branch they choose to staging or production servers (with permission check for the latter).
Yeah, we're that cool ;)

There's a bunch of techs involved here:

- Git server
- PHP app deployment
- Javascript app deployment
- Chatroom
- Bot

I'll go over each of these with workarounds/fixes for the main hurdles I encountered, but installation instructions will be *à la* [RTFM](http://en.wikipedia.org/wiki/RTFM). Be warned, some unix-fu and ssh-fu are required.

## Git server

You are using git, right?
Aside from making our life easier, git repos are where the deployment server takes the code from. No ftp'ing into the server and copying files manually (ohgodwhy) or rsync'ing. Less confusion and mixups.

If you don't want to get a Github account, you can set up your own Git server with [Gitolite](http://gitolite.com/).

#### Naming things

Before I continue, I'll just clear up what I mean for a couple of terms I'll use in the next parts.

- The "deployment server" is the server which runs the deployment commands. For ease of setup, I've pulled the repos of each project on this server, since each project has the instructions to deploy itself.
- "Prod and staging servers" are the (web)servers running the code/website.

## PHP app deployment

You know all the little things you usually do when deploying a new version of your app? run composer, set the right environment, run migrations, clear cache, etc…
[Rocketeer](http://rocketeer.autopergamene.eu/) can do all that for you, and more. It will version your deploys, and enable you to rollback if something goes awry. You can configure multiple "stages" (servers in staging or prod environment) and have access to callbacks for each step during the deployment.

I've used it successfully with [Laravel 4](http://laravel.com) (the project was originally a Laravel package) and [Yii 1.*](http://www.yiiframework.com/), more specifically with [Clevertech's YiiBoilerplate](https://github.com/clevertech/YiiBoilerplate) where deployment is a bit more complex than for the mint Yii.

#### A note on security

It's a good idea for the prod/staging servers to have a "deploy" user, as the deployment server will be accessing them and we don't want to give root access to an automatic tool.
How this impacts the deployment is that you usually restart some services afterwards and this requires sudo permissions!
Fear not, you can give sudo powers on specific commands to your deploy user. Here's for ubuntu server (12.04+ as far as I know):

```bash
$ sudo vi /etc/sudoers.d/deployer
```

If you want to give your deploy user ("deployer" here) permissions to restart nginx and fpm, add the lines:

```bash
deployer ALL=(ALL) NOPASSWD:/usr/sbin/service php5-fpm restart
deployer ALL=(ALL) NOPASSWD:/usr/sbin/service nginx restart
```

Another thing, use SSH keys to give access to your git repos and servers to your deploy user.
Little quirk for Laravel deployment, [you'll need an RSA key, not DSA](http://laravel.io/forum/03-15-2014-unable-to-connect-to-remote-server-with-ssh-key?page=1#reply-17102).

## Javascript app deployment

If you're building js apps, you probably know about [Grunt](http://gruntjs.com/) by now. I use grunt to minify and version css and js assets.

I've coupled it with [Shipit](https://github.com/shipitjs/shipit) to take care of deployment.
Because of the build required prior to the deploy and the way Shipit works, I had some trouble having the right files copied to the server. In short, Shipit checks out the repo into a "workspace" (a temporary directory) while the Grunt build was being run in the current directory. Solution:

```javascript
grunt.registerTask('workspacebuild', 'Build project in workspace', function () {
	grunt.shipit.local('cd ' + grunt.config('shipit.options.workspace')  +' && npm install && grunt build', this.async());
});

grunt.shipit.on('fetched', function () {
	grunt.task.run(['workspacebuild']);
});
```

Note: The build and deploy will be run from the deployment server, so it'll need to have Node.

The only bone I have with Shipit for now is that the deploy is quite slow at about ~7mins, and it's not even the build because that takes about 30s.

## Chatroom

If you don't already have one, getting a company chatroom is the next step. I can point you to [Hipchat](https://www.hipchat.com), basic features are free with unlimited users.
Running the deploys from a chatroom allows everyone to see what is going on, and not squash each other's deploys or have concurrent deploys (I don't even know how that would play out).

## Bot

The most fun part! Now that your projects have a deployment system, it's time to get a bot and explain to it how to run those deploys.

I use [Hubot](https://hubot.github.com/) (there are others). Hubot runs on Node, which is in my ballpark, and there is an official [Hipchat connector](https://github.com/hipchat/hubot-hipchat). I've set it up on the deployment server.

For the deployment command, if you have a Github repo, [you're in luck](https://github.com/atmos/hubot-deploy).

If not, creating your own deploy script is not so hard. Just give [Hubot's scripting guide](https://github.com/github/hubot/blob/master/docs/scripting.md) a read. Here is the major part of the script I wrote (you'll need [Hubot Auth](https://github.com/hubot-scripts/hubot-auth)):

```coffeescript
# list of projects and command to run the deploy per project
projects =
	laravelapp:
		type: "rocketeer"
		path: "/home/deployer/repos/laravelapp"
		command: 'php artisan deploy:deploy --on="#{target}" --branch="#{branch}" --quiet --no-interaction'
	jsapp:
		type : "shipit"
		path:"/home/deployer/repos/jsapp"
		command: 'grunt shipit:#{target} --branch="#{branch}" deploy &>/dev/null'

module.exports = (robot) ->
	run_cmd = (cmd, path, msg, completion) ->
		{spawn, exec}  = require 'child_process'
		exec cmd, {cwd : path}, (err, stdout, stderr) ->
			message = stdout
			if not stdout?
				message +=  "\r\n" + "Error: no stdout something went wrong"
			if err
				message +=  "\r\n" + "Error: " + err
				if stderr
					message +=  "\r\n" + "stderr:  " + stderr
			msg.send message
			msg.send completion

	robot.respond /deploy\ (\S+)\@(\S+)\ to\ (\S+)/i, (msg) ->
		# match[1]: project,	# match[2]: branch,	 match[3]: target
		if msg.match[3] == "prod"
			target = "production"
		else
			target = msg.match[3]

		branch = msg.match[2]
		project = msg.match[1]

		unless robot.auth.hasRole(msg.envelope.user, "deploy-" + target)
			msg.send ('you are not authorized to perform this deploy')
			return

		if projects[project] == undefined
			msg.send ('you sure about that project name?')
			return

		msg.send('deploying: {project: ' + project + ', branch: ' + branch + ', target: ' + target + ' }')
		cmd = projects[project].command.replace('#{target}', target).replace('#{branch}', branch)
		run_cmd cmd, projects[project].path, msg, 'deploy done: {project: ' + project + ', branch: ' + branch + ', target: ' + target + ' }'
```

Usage: `@[bot] deploy [project]@[branch] to [env]`
Giving your users deploy rights: `@[bot] Some Dude has deploy-[env] role`

## TL;DR

- Have a repo for each of your projects
- Add a deployment tool + deploy instructions per environment for each of your projects; pull onto your deployment server
- Set up a chat bot, connect it to your company chatroom
- Write a script to have your bot execute the deployment scripts from inside the project directories
- PROFIT

## Profit

With this setup, each developer is much more autonomous and we deploy all day long.

BTW, ever heard of [ChatOps](http://sneakycode.net/getting-started-with-chat-ops/)? I recently discovered the term myself, and have yet to read thoroughly about it.

Have fun with your deploys.
