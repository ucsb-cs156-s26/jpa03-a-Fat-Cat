# OAuth Setup

This Spring Boot application is set up to use Google OAuth as it's authentication scheme.

This document describes how to set up this app for OAuth on `localhost`.

Setting up dokku is described in [`/docs/dokku.md`](/docs/dokku.md), but *please follow these steps first*.

Before we get into the details, here's an overview of the process.  

1. The *first* time you set up Google OAuth for an application you will need to set up a *project* and configure the *OAuth consent screen* for that project.  This is where you set the limits on what access Google has to the user's information.  This is done at the [Google Developer Console](https://console.cloud.google.com/) and is explained below.
2. Every time you set up a Google OAuth application, you will need a Google  *client id* and *client secret*.  This is also 
   done at the [Google Developer Console](https://console.cloud.google.com/) and is also explained below.
3. Once you have these two values, you will put them in a `.env` file at the root of the repo; this is for only for testing the app on `localhost`.  The `.env` file is *not* checked into Github.

Now lets look at these steps in more detail. For some of the details, we'll refer you to article on the CMPSC 156 web site.

## First Time Only: Create project and Configure OAuth Consent Screen

If this is your first time setting up a Google OAuth application in this course, you may need to do three steps.
Later in the course, you'll only need to do the last of these, three, since the first two are typically "one-time" only steps.

* To set up a project in the Google Developer Console, follow these instructions:
   - <https://ucsb-cs156.github.io/topics/oauth/google_create_developer_project.html>
* To set up an OAuth Consent Screen for your project, follow these instructions:
   - <https://ucsb-cs156.github.io/topics/oauth/google_oauth_consent_screen.html>
 
## Obtain `GOOGLE_CLIENT_ID` and `GOOGLE_CLIENT_SECRET`

To create a set of OAuth credentials (`GOOGLE_CLIENT_ID` and `GOOGLE_CLIENT_SECRET` values), follow the instructions here:
* <https://ucsb-cs156.github.io/topics/oauth/oauth_google_setup.html>

## To run on localhost: configure `.env` file:


In the top level directory, use this command to copy `.env.SAMPLE` to `.env`.  Recall that you
may need to use `ls -a` to get the files to show up, since they are hidden files on Unix-like systems.

```
cp .env.SAMPLE .env
```

Recall that `.env` and `.env.SAMPLE` will not show up in regular directory listings; files starting with `.` are considered
hidden files.  Use `ls -a`, or configure your Mac finder/Windows explorer to show hidden files.

As explained below, put your client-id and client-secret into `.env`, NOT in `.env.SAMPLE` 
* `.env` is never committed to the GitHub repo
* There is more information about `.env` vs. `.env.SAMPLE` on this page if you are interested: [docs/environment-variables](environment-variables.md).

The file `.env.SAMPLE` **should not be edited;** and **should not be deleted from the repo**.  It is intended to
be a template for creating a file called `.env` that contains your repository secrets, and future developers will need it to stick around.

The `.env` is in the `.gitignore` because **a file containing secrets should NOT be committed to GitHub, not even in a private repo.

After copying, the file `.env` looks like this:

```
GOOGLE_CLIENT_ID=see-instructions
GOOGLE_CLIENT_SECRET=see-instructions
ADMIN_EMAILS=phtcon@ucsb.edu
```

Replace `see-instructions` with the appropriate values.

## Setting up `ADMIN_EMAILS` in `.env`

The `ADMIN_EMAILS` value is used to determine which users have access to administrative features in the app.  One of those
is the ability to list the users that have logged in.

For `ADMIN_EMAILS`, add your own email and any teammates you are collaborating with after phtcon.ucsb.edu; you can separate multiple emails with commas, e.g.

```
`ADMIN_EMAILS=phtcon@ucsb.edu,cgaucho@ucsb.edu,ldelplaya@ucsb.edu`
```

*Do not separate emails with spaces*; only commas:
* ❌ WRONG: `ADMIN_EMAILS=phtcon@ucsb.edu, cgaucho@ucsb.edu, ldelplaya@ucsb.edu`
* ✅ Correct: `ADMIN_EMAILS=phtcon@ucsb.edu,cgaucho@ucsb.edu,ldelplaya@ucsb.edu`

* Add your own UCSB email address
* Add `phtcon@ucsb.edu` (your instructor)
* Add the staff emails (look for these on the Slack help channel for this assignment)
* Add the emails of everyone else on your team

I suggest that, as a team, you collaborate in your team slack channel on getting a standard list of these, and then
that you pin that post in your team slack channel for easy reference.

With this done, you should be all set to run on localhost.


## Configuring dokku apps for OAuth


Before you try to configure a dokku app for OAuth, do these steps *in this order*.  If you try them in a different order, things won't work properly.

<b>Do not literally use <tt>jpa03-<i>yourGithubLogin</i></tt> in these instructions; substitute in your own github login in place of the words <tt><i>yourGithubLogin</i></tt></b>.

1. Configure the app on localhost using a `.env` file as explained above, and have the `.env` file open in an editor so that the values are handy.
2. Create a new dokku app with <tt>dokku apps:create jpa03-<i>yourGithubLogin</i></tt>
3. Set the app to ones that keeps the git directotry around:
   <tt>dokku git:set jpa03-<i>yourGithubLogin</i> keep-git-dir true</tt>
4. Use these commands to set up the configuration variables for the app.  Replace *value-from-env* in each case with the correct value from the .env file you configured for localhost:<br />
   <tt>dokku config:set --no-restart jpa03-<i>yourGithubLogin</i> PRODUCTION=true</tt><br />
   <tt>dokku config:set --no-restart jpa03-<i>yourGithubLogin</i> GOOGLE_CLIENT_ID=<i>value-from-env</i></tt><br />
   <tt>dokku config:set --no-restart jpa03-<i>yourGithubLogin</i> GOOGLE_CLIENT_SECRET=<i>value-from-env</i></tt><br />
   <tt>dokku config:set --no-restart jpa03-<i>yourGithubLogin</i> ADMIN_EMAILS=<i>value-from-env</i></tt><br />
5. Deploy and link a postgres database using<br />
   <tt>dokku postgres:create jpa03-<i>yourGithubLogin</i>-db</tt><br />
   <tt>dokku postgres:link jpa03-<i>yourGithubLogin</i>-db jpa03-<i>yourGithubLogin</i> </tt> <br />

   If you get a message like this, it is perfectly normal:
   ```
   -----> Restarting app jpa03-cgaucho
   !     App image (dokku/jpa03-cgaucho:latest) not found
   ```

   It just means that your app hasn't been started yet.
   Just continue with the instructions.

7. Build the app with regular `http` using the commands.  This will deploy an http only version of the app. When these commands complete, *you will still not be able to login yet, but you should be able to access the home page over `http`*:<br />
   <tt>dokku git:sync jpa03-<i>yourGithubLogin</i> https://github.com/ucsb-cs156-s26/jpa03-<i>yourGithubLogin</i> main</tt><br />
   <tt>dokku ps:rebuild jpa03-<i>yourGithubLogin</i></tt><br />
8. Now deploy https (encrypting) with these commands.
   This will deploy an https  version of the app.
   When these commands complete, *you should be able to login with OAuth*:<br />
   <tt>dokku letsencrypt:set jpa03-<i>yourGithubLogin</i> email <i>yourEmail</i>@ucsb.edu</tt><br />
   <tt>dokku letsencrypt:enable jpa03-<i>yourGithubLogin</i></tt><br />
   <tt>dokku ps:rebuild jpa03-<i>yourGithubLogin</i></tt><br />

For troubleshooting advice with OAuth, this page may help:

* <https://ucsb-cs156.github.io/topics/oauth/oauth_troubleshooting.html>

