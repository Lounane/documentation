---
title: v4 Plugin Migration - Migrate back end - Strapi Developer Docs
description:
canonicalUrl:
---

<!-- TODO: update SEO -->

# v4 plugin migration: Migrate the back end

!!!include(developer-docs/latest/update-migration-guides/migration-guides/v4/snippets/plugin-migration-intro.md)!!!

Migrating the back end of a plugin to Strapi v4 requires:

- updating [imports](#update-imports)
- updating content-types [getters](#update-content-types-getters) and, optionally, [relations](#update-content-types-relations)
- updating the [plugin configuration](#update-plugin-configuration)

<br/>

Some actions required to migrate the back end of a plugin can be performed by scripts that automatically modify code (codemods). The following table sums up the available options:

| Action                          | Migration type                                                                                               |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| Update imports                  | [Automatic](#update-imports-automatically) or [manual](#update-imports-manually)                             |
| Update content-types getters   | [Automatic](#update-content-types-getters-automatically) or [manual](#update-content-types-getters-manually) |
| Update content-types relations | [Manual](#update-content-types-relations)                                                                    |
| Update configuration            | [Manual](#update-configuration)                                                                              |

## Update imports

Package names in Strapi v3 are prefixed by `strapi-`.

Strapi v4 uses scoped imports.

To migrate to v4, update all Strapi imports from `strapi-package-name` to `@strapi/package-name`. Imports can be updated [automatically](#update-imports-automatically) or [manually](#update-imports-manually).

### Update imports automatically

:::prerequisites
!!!include(developer-docs/latest/update-migration-guides/migration-guides/v4/snippets/codemod-prerequisites.md)!!!
:::

::: caution
Codemods modify the plugin source code. Before running a command, make sure you have initialized a git repo, the working tree is clean, you've pushed your v3 plugin, and you are on a new branch.
:::

To update imports automatically:

- in the `package.json` file of the Strapi plugin, use the [`update-package-dependencies` codemod](https://github.com/strapi/codemods/blob/main/migration-helpers/update-package-dependencies.js) by running the following commands in a terminal:

  ```sh
  cd <the-folder-where-the-strapi-codemods-repo-was-cloned>
  node ./migration-helpers/update-package-dependencies.js <path-to-plugin>
  ```

- in any other file importing Strapi packages, use the [`update-strapi-scoped-imports` codemod](https://github.com/strapi/codemods/blob/main/transforms/update-strapi-scoped-imports.js) by running the following command in a terminal:

  ```sh
  npx jscodeshift -t ./transforms/update-strapi-scoped-imports.js <path-to-file | path-to-folder>
  ```

### Update imports manually

To update all imports manually, find any imports of Strapi packages (e.g. `strapi-package-name`) and rename them to `@strapi/package-name`.

## Update content-types getters

Strapi v3 models have been renamed to [content-types](/developer-docs/latest/development/backend-customization/models.md#content-types) in v4.

If the plugin declares models, update the syntax for all getters from `strapi.models` to `strapi.contentTypes`. The syntax can be updated [automatically](#update-content-types-getters-automatically) or [manually](#update-content-types-getters-manually).

### Update content-types getters automatically

::: caution
!!!include(developer-docs/latest/update-migration-guides/migration-guides/v4/snippets/codemod-modify-source-code.md)!!!
:::

To update the syntax for content-types getters automatically, use the [`change-model-getters-to-content-types` codemod](https://github.com/strapi/codemods/blob/main/transforms/change-model-getters-to-content-types.js) by running the following command in a terminal:

```jsx
npx jscodeshift -t ./transforms/change-model-getters-to-content-types.js <path-to-file | path-to-folder>
```

The codemod replaces all instances of `strapi.models` with `strapi.contentTypes` in the indicated file or folder.

### Update content-types getters manually

To update the syntax for content-types getters manually, replace any instance of `strapi.models` with `strapi.contentTypes`.

:::tip
Strapi v4 introduced new getters that can be used to refactor the plugin code further (see [Server API usage documentation](/developer-docs/latest/developer-resources/plugin-api-reference/server.md#usage)).
:::

## Update content-types relations

::: prerequisites
Updating content-types relations to v4 requires that the v3 models have been converted to v4 content-types (see [converting models to content-types documentation](/developer-docs/latest/update-migration-guides/migration-guides/v4/plugin/update-folder-structure.md#convert-models-to-content-types)).
:::

Strapi v3 defines relations between content-types with the `via`, `model` and `collection` properties in the model settings.

In v4, relations should be explicitly described in the `schema.json` file of the content-types (see [relations documentation](/developer-docs/latest/development/backend-customization/models.md#relations)).

<br />

If the plugin declares content-types with relations between them, migrating relations to v4 should be done manually in the [schema](/developer-docs/latest/development/backend-customization/models.md#model-schema) of the content-types.

To update content-type relations, update the `server/content-types/<contentTypeName>/schema.json` file for each content-type with the following procedure:

1. Declare the relation explicitly by setting the `type` attribute value to `"relation"`.

2. Define the type of relation with the `relation` property.<br/>The value should be a string among the following possible options: `"oneToOne"`, `"oneToMany"`, `"manyToOne"` or `"manyToMany"`.

3. Define the content-type target with the `target` property.<br/>The value should be a string following the `api::api-name.content-type-name` or `plugin::plugin-name.content-type-name` syntax convention.

4. (_optional_) In [bidirectional relations](/developer-docs/latest/development/backend-customization/models.md#relations), define `mappedBy` and `inversedBy` properties on each content-type.

::: details Example of all possible relations between an article and an author content-types

  ```json
  // path: ./src/plugins/my-plugin/server/content-types/article/schema.json
  
  // Attributes for the Article content-type
  "articleHasOneAuthor": {
    "type": "relation",
    "relation": "oneToOne",
    "target": "api::author.author"
  },
  "articleHasAndBelongsToOneAuthor": {
    "type": "relation",
    "relation": "oneToOne",
    "target": "api::author.author",
    "inversedBy": "article"
  },
  "articleBelongsToManyAuthors": {
    "type": "relation",
    "relation": "oneToMany",
    "target": "api::author.author",
    "mappedBy": "article"
  },
  "authorHasManyArticles": {
    "type": "relation",
    "relation": "manyToOne",
    "target": "api::author.author",
    "inversedBy": "articles"
  },
  "articlesHasAndBelongsToManyAuthors": {
    "type": "relation",
    "relation": "manyToMany",
    "target": "api::author.author",
    "inversedBy": "articles"
  },
  "articleHasManyAuthors": {
    "type": "relation",
    "relation": "oneToMany",
    "target": "api::author.author"
  }
  ```

  ```json
  // path: ./src/plugins/my-plugin/server/content-types/author/schema.json

  // Attributes for the Author content-type
  "article": {
    "type": "relation",
    "relation": "manyToOne",
    "target": "api::article.article",
    "inversedBy": "articleBelongsToManyAuthors"
  },
  "articles": {
    "type": "relation",
    "relation": "manyToMany",
    "target": "api::article.article",
    "inversedBy": "articlesHasAndBelongsToManyAuthors"
  }
  ```

:::

## Update plugin configuration

Strapi v3 defines plugin configurations in a `config` folder.

In v4, the default configuration of a plugin is defined as an object in the `config` property found in the `strapi-server.js` entry file (see [default plugin configuration documentation](/developer-docs/latest/developer-resources/plugin-api-reference/server.md#configuration)).

<br/>

To handle default plugin configurations in v4:

1. In the `strapi-server.js` file, create a `config` object.

2. Within the `config` object:
   - Define a `default` key that takes an object to store the default configuration.
   - Add a `validator` key, which is a function taking the `config` as an argument.

::: details Example of a default plugin configuration

  ```jsx
  // path: ./src/plugins/my-plugin/strapi-server.js

  module.exports = () => {
  // … bootstrap, routes, controllers, etc.
  config: {
      default: { optionA: true },
      validator: (config) => {
        if (typeof config.optionA !== 'boolean') {
          throw new Error('optionA has to be a boolean');
        }
      },
    },
  }
  ```

:::