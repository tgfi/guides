Protocol
========

A guide for getting things done.

Set up your laptop
-------------

Set up your laptop with [this script](https://github.com/thoughtbot/laptop)
and [these dotfiles](https://github.com/thoughtbot/dotfiles).

Install RVM
-------------

We use [Ruby Version Manager](http://rvm.io) to easily switch between rubies and gemsets.

    $ \curl -#L https://get.rvm.io | bash -s stable --autolibs=3 --ruby

Set up an existing Rails app
----------------

Get the code.

    git clone git@github.com:organization/app.git

Set up the app's dependencies.

    cd project
    ./bin/setup

For a Heroku project, use [Heroku config](https://github.com/ddollar/heroku-config) to get `ENV`
variables.

    heroku config:pull --remote staging

Delete extra lines in `.env`, leaving only those needed for app to function
properly. For example: `MERCHANT_ID` and `S3_SECRET`.

For other projects, create a `config/config.yml` file and see your project manager for values.

Use [Foreman](https://github.com/ddollar/foreman) to run the app locally.

    foreman start

It uses your `.env` file and `Procfile` to run processes just like Heroku's
[Cedar](https://devcenter.heroku.com/articles/cedar/) stack.


Tell your application what ruby version you are running in `.ruby-version`.

    ruby-1.9.3-p392

Tell your application what gemset you are running in `.ruby-gemset`.

    app

Let Pow know what ruby and gemset the app uses by creating a `.powrc` file.

    if [ -f "$rvm_path/scripts/rvm" ] && [ -f ".ruby-version" ] && [ -f ".ruby-gemset" ]; then
      source "$rvm_path/scripts/rvm"
      rvm use `cat .ruby-version`@`cat .ruby-gemset`
    fi


Maintain a Rails app
--------------------

* Avoid including files in source control that are specific to your
  development machine or process.
* Perform work in a feature branch.
* Delete local and remote feature branches after merging.
* Rebase frequently to incorporate upstream changes.
* Use a [pull request] for code reviews.

[pull request]: https://help.github.com/articles/using-pull-requests/

Write a feature
---------------

Create a local feature branch based off master.

    g co master
    g pull --rebase
    g create-branch your-initials-new-feature

Rebase frequently to incorporate upstream changes.

    g fetch origin
    g rebase origin/master

Resolve conflicts. When feature is complete and tests pass, stage the changes.

    rake
    g add -A
    g
    g ci -v

Write a [good commit message]. Example format:

    Present-tense summary under 50 characters

    * More information about commit (under 72 characters).
    * More information about commit (under 72 characters).

    http:://project.management-system.com/ticket/123

Share your branch.

    g push origin [branch]
<<<<<<< HEAD

Submit a [GitHub pull request].
=======
>>>>>>> Updating protocol guide with TGFI processes and home page.

Ask for a code review in [Campfire](https://campfirenow.com/).

Review code
-----------

A team member other than the author reviews the pull request. They follow
[Code Review](../code-review) guidelines to avoid
miscommunication.

They make comments and ask questions directly on lines of code in the Github
web interface or in Campfire.

For changes which they can make themselves, they check out the branch.

    g co <branch>
    rake db:migrate
    rake
    g diff staging/master..HEAD

They make small changes right in the branch, test the feature in browser,
run tests, commit, and push.

When satisfied, they comment on the pull request `Ready to merge.`

Merge
-----

Rebase interactively. Squash commits like "Fix whitespace" into one or a
small number of valuable commit(s). Edit commit messages to reveal intent.

    g rebase -i origin/master
    rake

View a list of new commits. View changed files. Merge branch into master.

    g log origin/master..[branch]
    g diff --stat origin/master
    g checkout master
    g merge-branch [branch
    g push

Delete your local and remote feature branch.

    g delete-branch [branch]

Deploy via Heroku
------

View a list of new commits. View changed files. Deploy to staging.

    g fetch staging
    g log staging/master..master
    g diff --stat staging/master
    g push staging master

Run migrations (if necessary).

    heroku run rake db:migrate -r staging

If necessary, run migrations and restart the dynos.

    heroku run rake db:migrate --remote staging
    heroku restart --remote staging

[Introspect] to make sure everything's ok.

    watch heroku ps --remote staging

Test the feature in browser.

Deploy to production.

    g fetch production
    g log production/master..master
    g diff --stat production/master
    g push production master
    heroku run rake db:migrate -r production
    heroku restart -r production
    watch heroku ps -r production

Watch logs and metrics dashboards.

Close pull request and comment `Merged.`

Deploy via Capistrano
------

View a list of new commits. View changed files. Deploy to staging.

If you are using heroku, you

    cap staging deploy

This will automatically run migrations (if necessary) and restart Passenger Phusion.

Test the feature in browser.

Deploy to production.

    cap production deploy

Watch logs and metrics dashboards.

Close pull request and comment `Merged.`

Set Up Production Environment
-----------------------------

* Make sure that your
  [`Procfile`](https://devcenter.heroku.com/articles/procfile)
  is set up to run Unicorn.
* Make sure the PG Backups add-on is enabled.
* Create a read-only [Heroku Follower] for your production database. If a Heroku
  database outage occurs, Heroku can use the follower to get your app back up
  and running faster.

[Heroku Follower]: https://devcenter.heroku.com/articles/improving-heroku-postgres-availability-with-followers
[`Procfile`]: https://devcenter.heroku.com/articles/procfile
