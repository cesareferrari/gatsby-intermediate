# Instructions for working with Gatsby Themes

Create a `packages` directory at the top level and inside of it create a
directory for the theme, in this case `gatsby-theme-docs`.
It's mandatory to name a theme starting with `gatsby-theme-<theme-name>`

```
mkdir packages
mkdir packages/gatsby-theme-docs
```

Inside the theme directory, run `yarn init`. Fill out the questions, or accept
defaults. This will create a package.json file with this content:

```
{
  "name": "gatsby-theme-docs",
  "version": "1.0.0",
  "main": "index.js",
  "license": "MIT"
}
```

`gatsby-theme-docs` is the name of the yarn workspace.

To create workspaces we go to the theme root and create a package.json file.

```
touch package.json
```

Inside the file, add this code. Set private to true, because yarn workspaces
need to be set this way.

List the workspaces in an array, including everything inside packages, and
sites.

```
{
  "private": true,
  "workspaces": [
    "packages/*",
    "sites/*"
  ]
}
```

Now if we go inside the theme folder and issue `yarn workspaces info` we see the
list of all workspaces, including the theme folder:

```
yarn workspaces v1.16.0
{
  "gatsby-theme-docs": {
    "location": "packages/gatsby-theme-docs",
    "workspaceDependencies": [],
    "mismatchedWorkspaceDependencies": []
  },
  "honkify": {
    "location": "sites/honkify",
    "workspaceDependencies": [],
    "mismatchedWorkspaceDependencies": [
      "honkify"
    ]
  },
  "lookup": {
    "location": "sites/lookup",
    "workspaceDependencies": [],
    "mismatchedWorkspaceDependencies": []
  },
  "negronis": {
    "location": "sites/negronis",
    "workspaceDependencies": [],
    "mismatchedWorkspaceDependencies": []
  }
}
```

Inside the theme folder, create an empty `index.js` file that is required by
yarn.


Create a new site folder inside `sites` called `theme-dev`

```
cd sites
mkdir theme-dev
```

Inside `theme-dev` run `yarn init` and set the package to private in the
questions asked:

```
{
  "name": "theme-dev",
  "version": "1.0.0",
  "main": "index.js",
  "license": "MIT",
  "private": true
}
```

To add packages to a workspace, instead of using just `yarn add`, we need to
prefix the workspace name (in this case `theme-dev` like below.

Inside theme-dev directory we add gatsby, react and react-dom

```
yarn workspace theme-dev add gatsby react react-dom
```

We then add the theme in the same way (which is gatsby-theme-docs we have created before).
We need to add `@*` at the end so Node won't complain that the theme package is
private.

```
    yarn workspace theme-dev add "gatsby-theme-docs@*"
```

## Create Gatsby config

Now inside the theme-dev folder, create a `gatsby-config.js` file and add the
theme we want to use to it, in the plugins array:

```
module.exports = {
  plugins: ['gatsby-theme-docs']
}
```

In the package.json file (inside theme-dev) add the script to start the
development environment:

```
 "scripts": {                                                                 
    "develop": "gatsby develop"                                                
  },       
```

If we now run the script with yarn, it will start the gatsby site (it will give
a 404 error because we have no pages yet).

```
yarn workspace theme-dev develop
```
