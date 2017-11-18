# How to enable Travis in a repository under the Node.js Github organization

Since we don't allow third-party access to the Node.js Github organization
except our own bots, when a repository is transferred into the Node.js
foundation, Travis won't be able to update the build status of Pull requests
automatically. Instead, we need to use our own github bot to pull the status
from Travis and send the update it Github. This is what needs to be done:

### Step 1: Travis setup

Make sure Travis can listen to the push and pull request events of the
repository. Go to "Settings -> Integrations & services" page of the
repository. If it has previously configured Travis, then Travis should be
listed under "Services". If not, click on the dropdown "Add service", and
search for Travis to add it.

The service does not really have to be configured to enable travis. All of the
following configurations works:

* User and Token configured as someone who is not "nodejs", and Domain is
  blank
* User and Token left blank, Domain is `notify.travis-ci.org`
* User and Token configured as someone who is not "nodejs", Domain is
  `notify.travis-ci.org`
* User, Token, Domain all left blank
  
If `https://travis-ci.org/nodejs/{reponame}` starts building new pull requests
or pull request updates in the repository after the service is enabled, then
the Travis end is set up.

### Step 2: add the bot to your team

Go to the "Settings -> Collaborators & teams" page of the repository, add
`@nodejs-github-bot`, or the `@nodejs/bots` team to the collaborators of the
repository, so that the Github bot account has write access to it. If there
are 404 responses in the logs of build status updates sent from the bot, it
usually means the bot is not granted write access to the repository. 

### Step3: add the webhook for the bot

On the "Settings -> Webhooks" page, add a new webhook for the bot:

* Payload URL: `http://github-bot.nodejs.org:3333/hooks/github`
* Content type: `application/json`
* Secret: can be obtained in the https://github.com/nodejs/secrets repository,
  or ask someone from the build working group to add the webhook for you.
* In "Which events would you like to trigger this webhook?", select "Let me
  select individual events.", check "Push" and "Pull Request".
* Finally, check "Active" and save the webhook. Github would send a test
  payload to the bot, if it came back green, the bot is set up with the
  repository.

### Step 4: update the bot

Open a pull request to https://github.com/nodejs/github-bot , adding the
repository to the `enabledRepos` in
https://github.com/nodejs/github-bot/blob/master/scripts/display-travis-status.js,
so the bot would not ignore the events from your repository ans start sending
updates.

### How it works

The bot works like this:

1. when a pull request has been opened or updated,
Github will send an event to both Travis (configured in step 1) and the bot
(step 3).
2. Travis would start the build, and the bot would start polling Travis
for build status (this requires step 4 otherwise the bot would ignore the
event).
3. When the bot find the build matching the last commit in the PR, it
would update the build status of that PR as "pending" (this update requires
step 2).
4. When a poll came back as a successful build, the bot would update the
build status as "success".

### Aside

If the transferred project is used across different platforms, consider
transferring the testing to the Node.js build infrastructure as well, so that
it can be tested across the platforms supported by Node.js. Open an issue in
https://github.com/nodejs/build for help getting the transfer done.
