---
title: Setting Up Postgres on MacOS for Development
---

Setting Up Postgres on macOS for Development
==============================================

This is a simple guide to help you set up a postgres database service on your
mac. This will allow to develop. This guide assumes you have homebrew installed.
If you don't, do that first [here](brew.sh).

## Install Postgres

Simple if you have homebrew.

```
$ brew install postgresql
```

Once the install is complete it will ask you a couple question to set it up.
Username, password... etc. 

After the initialization is complete, you may have noticed mention of brew
services for running postgres as a background service. This will be convenient
to automatically start postgres at system startup, and easily start and stop the
process.

```
$ brew tap homebrew/services
```

Now we can start the postgres service with the following command,

```
$ brew services start postgresql
# ==> Successfully started `postgresql` (label: homebrew.mxcl.postgresql)
```

It can be stopped,

```
$ brew services stop postgresql
# Stopping `postgresql`... (might take a while)
# ==> Successfully stopped `postgresql` (label: homebrew.mxcl.postgresql)
```

or it can be restarted,

```
$ brew services restart postgresql
# Stopping `postgresql`... (might take a while)
# ==> Successfully stopped `postgresql` (label: homebrew.mxcl.postgresql)
# ==> Successfully started `postgresql` (label: homebrew.mxcl.postgresql)

```

Donezo! Now you can use postgres in your application development locally.
In the setup use your user's username as the user for the postgres db.
