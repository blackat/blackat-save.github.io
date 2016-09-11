---
layout: post
title: "Furure of Declaration Files"
date: 2016-09-11 11:15:37 +0200
comments: true
categories: TypeScript, typings, Angular2
---
When I have started to use TypeScript I have looked at [Definitely Typed](http://definitelytyped.org/) to see how many  people where writing declaration files (`.d.ts`) and then [guidelines from Microsoft](https://typescript.codeplex.com/wikipage?title=Writing%20Definition%20%28.d.ts%29%20Files) to see how to do it. There was then the utility [tsd](https://github.com/DefinitelyTyped/tsd), but now many stuff has changed.

<!-- more -->
## Evolution at Glance
### TSD Package Manager
{% blockquote DefinitelyTyped https://github.com/DefinitelyTyped/tsd TSD %}
TSD is a package manager to search and install TypeScript definition files directly from the community driven DefinitelyTyped repository.
{% endblockquote %}

A tool has to be installed via `npm` to fetch the package and start to configure a file named `tsd.json` that holds all the declaration (or definition) files. Basically `tsd` is like `npm` with similar commands able to manage defintion dependencies used by TypeScript compiler to transpile `.ts` files and by type guard to perform runtime check.

{% blockquote TypeScript https://www.typescriptlang.org/docs/handbook/advanced-types.html Documentation %}
A type guard is some expression that performs a runtime check that guarantees the type in some scope.
{% endblockquote %}

### Typings
Some months go, looking at the [GitHub repository](https://github.com/DefinitelyTyped/tsd), I have discovered [the project has been deprecated](https://github.com/DefinitelyTyped/tsd/issues/269) in favor of [typings](https://github.com/typings/typings). So `typings` is an evolution of `tsd` with some advantages explained [here](https://github.com/typings/typings/issues/72) by [blakeembrey](https://github.com/blakeembrey).

Basically the previous tool has been improved into a more generic package manager that acts like `npm`, then it still requires to be installed. Old projects have to be updated to migrate from `tsd` if previously used. Slighter different syntax needs to be learnt.

### @Types
Recently I was experimenting [Angular2 Command Line Interface](https://github.com/angular/angular-cli) and I have noticed that generating a new project the folder `typings` was not created, something missing? Not at all, TypeScript 2.0 has improved once again and a new property `types` has been kicked in the `tsconfig.json`.

So looking at the `tsconfig.json` [schema definition](http://json.schemastore.org/tsconfig) we can find
```json
types: {
  "description": "Type declaration files to be included in compilation. Requires TypeScript version 2.0 or later.",
  "type": "array",
  "items": {
    "type": "string"
  }
}
```
The evolution is explained in this post from [Daniel Rosenwasser](https://blogs.msdn.microsoft.com/typescript/2016/06/15/the-future-of-declaration-files/) Program Manager on TypeScript.

In a nutshell
{% blockquote Daniel Rosenwasser https://blogs.msdn.microsoft.com/typescript/2016/06/15/the-future-of-declaration-files The future of declaration files %}
Getting type declarations in TypeScript 2.0 will require no tools apart from npm.
{% endblockquote %}

Really a good news! we can rid off some extra entities in our project such as:

- another package managers then `npm`;
- `json` file for type declarations;
- configure `tsconfig.json` to exclude files from `typings` folder;
- the famous `///reference` at the beginning of each `.ts` file.

#### Usage
Very simple:
```bash
npm install --save @types/lodash
```

{% blockquote Daniel Rosenwasser https://blogs.msdn.microsoft.com/typescript/2016/06/15/the-future-of-declaration-files The future of declaration files %}
From there you’ll be able to use lodash in your TypeScript code with no fuss. This works for both modules and global code.
{% endblockquote %}

No more additional configuration or references at the beginning of each file.

#### Type Search
Please refer to [TypeSearch](http://microsoft.github.io/TypeSearch/) search engine to find a type definition and how to install it via `npm`.

At first sight I could say the project is cleaner, less utilities and, may be, a more consistent way to manage `.d.ts` files. I hope a definitive path has been defined (at least for a couple of years) in order to have all the dependencies centralized in one place.
