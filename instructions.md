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

## Prebootstrap

Go into the theme folder and create a `gatsby-node.js` file.

Also, create a `utils` folder and a `default-options.js` file inside of it

```
touch gatsby-node.js
mkdir utils
touch utils/default-options.js
```

In the default-options.js file, add this code:

```
module.exports = ({
  basePath = '/',
  contentPath = 'docs',
  useExternalMDX = false
}) => ({basePath, contentPath, useExternalMDX})
```

In the `gatsby-node.js` file, add this code:

```
export.onPreBootstrap = ({store}, options) => {
  const {program} = store.getState()getState()
}
```

Install the mkdirp package, to create directories:
```
yarn workspace gatsby-theme-docs add mkdirp
```

Add this code to `gatsby-node.js`:

```
const path = require('path');
const fs = require('fs');
const mkdirp = require('mkdirp');
const withDefaults = require('./utils/default-options');

exports.onPreBootstrap = ({store}, options) => {
  const { program } = store.getState();
  const { contentPath } = withDefaults(options);
  const dir = path.join(program.directory, contentPath);

  if (!fs.existsSync(dir)) {
    mkdirp.sync(dir)
  }
}
```

If we start the development server in theme-dev, it should create the docs
folder that was specified in the options:

```
yarn workspace theme-dev develop
```

## Create MDX index.mdx file in docs folder in theme-dev

In the `docs` folder that was created by the theme, we add a index document in
MDX format.

```
# in theme-dev
touch docs/index.mdx
```

Content of the `index.mdx` file:

```
---
title: home
---

These are docs
```

To show this file we need to install `gatsby-source-filesystem` plugin in the
theme (using yarn workspaces):

```
yarn workspace gatsby-theme-docs add gatsby-source-filesystem gatsby-plugin-mdx
@mdx-js/mdx @mdx-js/react
```

the plugin allows to use a folder in the project to use as a source of data by
putting it in the graphql layer.

`gatsby-plugin-mdx` looks for any file that are MDX and convert them into mdx
nodes inside Graphql

`gatsby-plugin-mdx` has dependency on `@mdx-js/mdx` and `@mdx-js/react` so we
need to install those as well

After installing the plugins, we need to configure them in `gatsby-config.js`
for the theme:

```
touch packages/gatsby-theme-docs/gatsby-config.js
```

Since we are in a theme, gatsby-config.js needs to export a function that takes options, instead of merely a config object.

The general structure is this:

```
module.exports = options => { # return config object }
```

We pass the options in from the utils/default-options.js file we have set up
previously.
We import the file and use the options we need:

```
const withDefaults = require('./utils/default-options');

module.exports = options => {
  const { contentPath, useExternalMDX } = withDefaults(options);

  return {
    plugins: [
      {
        resolve: 'gatsby-source-filesystem',
        options: { name: 'gatsby-theme-docs', path: contentPath },
      },
    ],
  };
};
```

In the `gatsby-source-filesystem` plugin configuration, we use a name of the
theme, so if another theme uses MDX we are able to discriminate which MDX files
we need to grab. MDX files for this theme have the `gatsby-theme-docs` name.
For the path, we use the configuration options e have defined in the utils file.

Next, we need to set up the gatsby-plugin-mds options but only if
`useExternalMDX` is not defined. `useExternalMDX` can be defined in the main
site, so in that case we won't use the options from the theme. If it's not
defined in the main site (the site that uses this theme), then we use the ones
in the theme.

This can be done using this pattern:

```
// if useExternalMDX is false, evaluate what is at right of &&

      !useExternalMDX && {...}
```

```
      !useExternalMDX && {
        resolve: 'gatsby-plugin-mdx',
        options: {
          defaultLayouts: {
            default: require.resolve('./src/components/layout.js')
          }
        }
      }
```

Since there may be a chance that `!useExternalMDX` is false, we need to filter
the plugins array for Booleans, so it drops any false item. That way Gatsby won't
complain for a false item in the array.

```
plugins: [ ...  ].filter(Boolean),
```

This is the final code in the theme `gatsby-config.js`:

```
const withDefaults = require('./utils/default-options');

module.exports = options => {
  const { contentPath, useExternalMDX } = withDefaults(options);

  return {
    plugins: [
      {
        resolve: 'gatsby-source-filesystem',
        options: { name: 'gatsby-theme-docs', path: contentPath },
      },
      !useExternalMDX && {
        resolve: 'gatsby-plugin-mdx',
        options: {
          defaultLayouts: {
            default: require.resolve('./src/components/layout.js')
          }
        }
      }
    ].filter(Boolean),
  };
};
```

## Create the layout file

Note that it's not necessary to install `react` and `react-dom` in the theme
directory, since the site that uses this theme will have `react` installed
anyway.

Inside `packages/gatsby-theme-docs/src/components/layout.js` add this code:

```
import React from 'react';

const Layout = ({ children }) => (
  <React.Fragment>
    <header>gatsby-theme-docs</header>
    <main>{children}</main>
  </React.Fragment>
);

export default Layout;
```

Now if we start the `theme-dev` site with `gatsby develop` and look at the
graphql layer with graphiql we can run a query and see the home page we put
there in the previous step:

```
query MyQuery {
  allMdx {
    nodes {
      frontmatter {
        title
      }
    }
  }
}
```


## Schema customization
Purpose of schema customization is to customize grapql queries (more on this
later).

We do schema customization using the schema customization api of Gatsby called
`createSchemaCustomization`.


In `packages/gatsby-theme-docs/gatsby-node.js` add this code:

```
exports.createSchemaCustomization = ({actions}) => {
  actions.createTypes(`
    type DocsPage implements Node @dontInfer {
      id: ID!
      title: String!
      path: String!
      updated: Date! @dateformat
      body: String!
    }
  `)
}
```

`createSchemaCustomization` takes some actions and the action we need is
`createTypes`. `createTypes` creates custom types inside the graphql layer.
These types we can define to do what we want.
Since these are docs pages, we create a type of `DocsPage`. `DocsPage`
implements the `Node` type, which is the core type for data. This tells Gatsby
that `DocsPage` is a `Node` so we can get the Node properties with it, like
filtering, querying on field name and so on.

`@dontInfer` tells Gatsby to not infer other fields automatically.

Next we specify what fields we want for the type:

```
id: ID!
title: String!
```

ID and String are the graphql types and the ! marks them as being required.

```
updated: Date! @dateformat
```
`@dateformat` is a Gatsby helper that signifies that we would need to format the
date

Now if we start the development server on the site that includes the theme
(theme-dev) and we look at Graphiql, we can see a custom type of `allDocsPage`
and `DocsPage`.

Looking at `allDocsPage > node` we can see the types we have specified, like
`path`, `title`, `updated`. If we click on `updated` we see the options to
format the date.

## Create MDX nodes

At this point, we need to listen to any MDX nodes that are created in the
theme-dev site and create a DocsPage out of them.

We add another API hook to `gatsby-node.js` (in the theme). It's called
`onCreateNode`. Parameters passed are
- node: the node that was created
- actions: an object full of actions that can help process this node
- getNode: ability to load the content of a given node
- createNodeId: helper that gives us the node id
- options 

Inside the function, we need the `basePath` that we pull out of the default
options.
We also get the node parent.

Next we make sure we only work on MDX files loaded by this theme. We do that
with an if statement. If the node type is not Mdx or the parent is not the theme
(as identified by the theme name) we return and stop processing this node.

Next we check for the parent name. If it's not index, we use it, otherwise we
leave it blank.
This is to identify the index page and load it at the '/docs' url instead of '/docs/index'

Next we call the createNode action that accepts the new node.

Inside we create the node id, which needs to be unique. In this case we use the
node type and the node id.
The title should come from the frontmatter, but if it's not set it will use the
parent file name. (remember, the parent is the file from which this node is
created.)

This is the code for this functionality:

```
exports.onCreateNode = ({ node, actions, getNode, createNodeId }, options) => {
  const { basePath } = withDefaults(options);
  const parent = getNode(node.parent);

  if (
    node.internal.type !== 'Mdx' ||
    parent.sourceInstanceName !== 'gatsby-theme-docs'
  ) {
    return;
  }

  const pageName = parent.name !== 'index' ? parent.name : '';

  actions.createNode({
    id: createNodeId(`DocsPage-${node.id}`),
    title: node.frontmatter.title || parent.name,
    updated: parent.modifiedTime,
    path: path.join('/', basePath, parent.relativeDirectory, pageName)
  })
};
```
