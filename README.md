# Arooo - A Room Of One's Own
[![Build Status](https://travis-ci.org/doubleunion/arooo.svg)](https://travis-ci.org/doubleunion/arooo)
[![Open Source Helpers](https://www.codetriage.com/doubleunion/arooo/badges/users.svg)](https://www.codetriage.com/doubleunion/arooo)

## Welcome :rocket::rocket::rocket:✨✨

This is a membership application app written by members of [Double Union](http://doubleunion.org/), a feminist hacker/makerspace for women in San Francisco.

This app is named after a famous Virginia Woolf essay, A Room of One's Own. You can learn more about it [on Wikipedia](http://en.wikipedia.org/wiki/A_Room_of_One's_Own)!

Also, here is a puppy that is saying "arooo":

[![A puppy howling](http://cdn3.sbnation.com/assets/2875387/v728228.gif)](https://www.youtube.com/watch?v=2Tgwrkk-B3k)

This application has a lot of cool features, including:
* Prospective members can apply for membership
* Current members can vote and comment on applications
* Current members can see a directory of members
* Current members can pay dues via Stripe
* Membership coordinators can manage member status

The application supports three levels of membership: members, key members, and voting members, where any member can see and comment on an application, but only voting members can vote. Membership coordinators can set whether the app is accepting applications, accept or reject individual applications, manage membership levels, and review dues status.

It is currently very Double Union specific, but we'd like to extract the Double Union things out of it, so that other feminist hacker/makerspaces can use it, too! We open sourced the app so that we can work with other hacker/makerspaces on that process. This app supports an application process that helps us maintain a safe space for our members, and we want this app to help other groups that have the same goal. If you're interested in using Arooo for your org, feel free to use it as-is or fork it and do that! We would like to make arooo more reusable and all help is welcome :) Feel free to [make an issue](https://github.com/doubleunion/arooo/issues/new) to discuss :) :dancer:

To check out a couple of screenshots, [see our Arooo announcement post](https://doubleunion.tumblr.com/post/101099822404/meet-arooo-a-open-source-membership-management).


## How to Contribute

We use [GitHub issues](https://github.com/doubleunion/arooo/issues) for feature development and bug tracking.

Anyone is welcome to make an issue or a pull request. We would *love* for first-time contributors to pick one of our [good first issue](https://github.com/doubleunion/arooo/issues?q=is%3Aissue+is%3Aopen+label%3A%22good+first+issue%22) issues :) 

We have a mailing list! Feel free to ask any question, including basic git and ruby questions etc :) https://groups.google.com/a/doubleunion.org/forum/#!forum/public-du-code

### Github flow

If you are new to GitHub, you can [use this guide](http://railsbridge.github.io/bridge_troll/) for help making a pull request.


### Development setup

Do the below OR if you prefer docker, see the Docker Setup section 

1. install a ruby version manager: [rvm](https://rvm.io/) or [rbenv](https://github.com/rbenv/rbenv)
1. when you cd into the project diretory, let your version manager install the ruby version in `.ruby-version`
1. `gem install bundler`
1. Fork the repo (click the Fork button above), and clone your fork to your local machine. [Here's a GitHub tutorial](https://help.github.com/articles/fork-a-repo/)
1. Make sure that postgres is installed [brew install postgres](https://wiki.postgresql.org/wiki/Homebrew) OR brew postgresql-upgrade-database (if you have an older version of postgres)
1. `bundle install`
1. `cp config/database.example.yml config/database.yml`
1. `cp config/application.example.yml config/application.yml`
1. `rake db:test:prepare` Now you can write and run tests! You can skip the other setup steps until you want to run arooo locally. :)
1. `bundle exec rake spec # runs tests` 
1. `bundle exec standardrb --fix # auto-fix linting issues (optional)` [more linter info](https://github.com/testdouble/standard)
1. `bundle exec rake db:setup # requires running local postgres`
1. `rails db:migrate`
1. `bundle exec rake populate:users # Populate data in your local database - optional`
1. `bundle exec rails server` # run server
1. `bundle exec rails console` # run console (useful for looking at and changing your local data)

#### Docker setup (optional)

1. Install docker and docker compose

1. Duplicate db config
```cp config/database.docker.yml config/database.yml```

1. build 
```docker-compose build```

1. build 
```docker-compose run --rm app bash -c bundle```

1. setup DB
```docker-compose run --rm app bundle exec rake db:setup```

#### Set up an application for local OAuth: 

1. Github
    * http://github.com/settings/applications/new
    * Application name: Whatever you want
    * Homepage URL: http://localhost:3000
    * Authorization callback URL: http://localhost:3000/auth/github/callback
    * in config/application.yml set `GITHUB_CLIENT_KEY` and `GITHUB_CLIENT_SECRET` to the Client ID and
      Client Secret from your Github application
    * Don't forget to restart your Rails server so it can see your shiny new GitHub key & secret
1. Google
    * TODO: figure this out and write it down 

#### Common errors

1. If you see the error `FATAL: role “postgres” does not exist`, if you are on OSX with brew run `/usr/local/Cellar/postgresql/<version>/bin/createuser -s postgres`

### Tests

Tests, also known as specs, are great! Adding tests is a great pull request all on its own. Please try to write tests when you add or change functionality. 

Run `rake db:test:prepare` after you pull or make any changes to the app, generally.

Make sure `bundle exec rails spec` passes before pushing your changes. (Our TravisCI integration will double-check before we merge code, so it's ok if you forget sometimes) :)

If you want to add a new kind of tests, or refactor the existing tests, do it!

### User states

The current User state machine can be found in `app/models/user.rb` It is the main moving piece of the
application.

Valid states:

* `visitor`: default state, no admin access, no application access
* `applicant`: not yet a member, no admin access, can only
  view/edit/save/submit their application
* `member`: access to member admin section, cannot vote
* `key_member`: access to member admin section, cannot vote, has keys to the space
* `voting_member`: access to member admin section, can vote, has keys to the space


#### Manually changing a user's state

First, open a Rails console in a terminal window, from the same directory as the app:

```
rails console
```

Now you can update any user:

```
> user = User.find_by_username('someusername')
> user.state # => "visitor"

> user.make_applicant!  # => true
> user.make_member!     # => true
> user.make_key_member! # => true
> user.make_voting_member! # => true
> user.make_applicant!  # => raises invalid state transition error

> user.update_attribute(:state, 'applicant') # bypasses normal checks & succeeds
```

If you need to make or unmake an admin, have a current admin click the un/make admin button on a member in the Member Admin View. Admins can accept/reject applications, update any member's status, see current member's dues, open and close applications, and manage new member setup.

## Production maintainer / SRE guide

### Rails console - heroku

You only need this if you are deploying code, checking changes, or maintaining a production instance of arooo

Set up heroku commandline client: https://devcenter.heroku.com/articles/heroku-cli

Staging: `$ heroku run rails console --remote staging`

Production: `$ heroku run rails console --remote production`


### Bugsnag
[www.bugsnag.com](https://www.bugsnag.com) is a heroku plugin that records errors in the production app. This is helpful for debugging. For bugsnag access, ask someone with access to the board@ section of 1Password to log into bugsnag and send you an email invite to create an account.
Thank you to Bugsnag for their [OSS program](https://www.bugsnag.com/open-source) :) ![bugsnag logo](https://global-uploads.webflow.com/5c741219fd0819540590e785/5c741219fd0819856890e790_asset%2039.svg)



### Deploying and Heroku access

This section only pertains if you have heroku & deployment access

If you are a DU member, see https://docs.google.com/document/d/19LbIYB2RDy-17UXuQx6wLgKp2EdLdqj-pg1cm3EpSb8/edit for more information on getting permission.

1. Add Heroku remotes to your `.git/config` (type `git remote --help` for more instructions on how to configure git remote.)

Note: Only maintainers have heroku access and can deploy.

  ```
  [remote "production"]
     url = git@heroku.com:du-arooo.git
     fetch = +refs/heads/*:refs/remotes/heroku/*
  [remote "staging"]
     url = git@heroku.com:doubleunion-staging.git
     fetch = +refs/heads/*:refs/remotes/heroku/*
  ```

1. Pull down the latest code from `master`

  ```
  git checkout master
  git pull --rebase origin master
  ```

1. If Travis CI tests are passing, push to the `staging` environment

  ```
  git checkout master
  git pull --rebase origin master
  git push staging master
  ```

1. If needed, perform rake tasks or set ENV variable settings on `staging`
1. Test [staging](http://doubleunion-staging.herokuapp.com/)!

  ```
  username: doubleunion
  password: meritocracyisajoke
  ```

1. After confirming that the code works on `staging`, push it to `production`!

  ```
  git checkout master
  git pull --rebase origin master
  git push production master
  ```

1. If needed, perform rake tasks or set ENV variable settings on `production`


### Email

This app sends emails, but who is our email provider? TODO


#### Staging

Currently neither github nor google auth works on staging- we should get this working again so we can actually test. 

The basic-auth login is found in https://dashboard.heroku.com/apps/doubleunion-staging/settings under BASIC_AUTH_NAME/BASIC_AUTH_PASSWORD

### Email

This app sends emails, but who is our email provider? TODO


## Security

If you find a security vulnerability with Arooo, please let us know at admin@doubleunion.org. Thank you!

## License

Copyright (C) 2014 Double Union

This program is free software: you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation, either version 3 of the License, or (at your option) any later version.

This program is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.

See the LICENSE.txt file for the full license.
