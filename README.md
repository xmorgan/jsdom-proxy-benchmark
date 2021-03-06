# Benchmark for proxies in Node.js

## Introduction

This benchmark compiles the [ECMAScript specification](https://github.com/tc39/ecma262) using the [ecmarkup](https://github.com/bterlson/ecmarkup) tool.

Ecmarkup uses [jsdom](https://github.com/tmpvar/jsdom) to manipulate its input and produce its output. jsdom allows construction of an in-memory representation of the DOM given an input HTML string (such as the ECMAScript specification source code). This DOM representation is meant to faithfully follow that of browsers.

As of [v11.2.0](https://github.com/tmpvar/jsdom/blob/master/Changelog.md#1120) of jsdom, it uses proxies to implement the very common data structures `NodeList` and `HTMLCollection`. These proxies can be seen in the following files:

- `node_modules/jsdom/lib/jsdom/living/generated/HTMLCollection.js`
- `node_modules/jsdom/lib/jsdom/living/generated/NodeList.js`

These proxies implement the following traps:

- `get`
- `has`
- `ownKeys`
- `getOwnPropertyDescriptor`
- `set`
- `defineProperty`
- `deleteProperty`
- `preventExtensions`

The algorithms used for these traps are precisely those given in the [Web IDL specification](https://heycam.github.io/webidl/) for [legacy platform objects](https://heycam.github.io/webidl/#es-legacy-platform-objects). Note that `HTMLCollection` has both indexed and named properties, while `NodeList` has only indexed properties. Both only have indexed/named property getters, not setters or deleters.

## How this repository is set up

All necessary code to run the benchmark is checked in to this repository, for maximum stability.

### Update steps

- To update the benchmark with new versions of ecmarkup and jsdom, run `npm install` and commit the results.
- To update the benchmark with a new copy of the ECMAScript specification, run `./scripts/download-ecma262.sh`, then `node ./scripts/inputs-to-js.js`, and commit the results.

### The benchmark itself

To run the benchmark, run `node ./benchmark/benchmark.js`.

A copy of the output from the benchmark is already committed as `benchmark/output-for-verifying.js`. The benchmark uses this to ensure no dead code elimination happens, although I suppose that would be unlikely given the potential for side effects.

## Potential future work

We will likely port another commonly-used class, `DOMTokenList`, over to use proxies in [tmpvar/jsdom#1953](https://github.com/tmpvar/jsdom/pull/1953).

As mentioned in [tmpvar/jsdom#1934](https://github.com/tmpvar/jsdom/pull/1934), converting `NamedNodeMap` (used for all attribute access) over to use proxies causes compilation time to increase by 6.7x. We hope to get ahold of this code and store it in a branch or something so that it can be compared to the current version, and optimized until the slowdown is eliminated.

We may be able to package this benchmark to run in a browser as well as in Node.js, since [jsdom can be run inside a web browser](https://github.com/tmpvar/jsdom#running-jsdom-inside-a-web-browser). We have already started down this path by writing the benchmark to use Ecmarkup APIs instead of the command-line tool, and storing the input as a `.js` file instead of a `.html` file. In fact, this almost works; some artifacts are in the repository already. But the result has a JavaScript error that I haven't yet had time to debug.
