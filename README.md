# Demo to reproduce a bug

Submitted as an issue to create-react-app here:
https://github.com/facebook/create-react-app/issues/12365

### Describe the bug

When running a TypeScript project created with the latest create-react-app (5.0.1), TypeScript compile errors will not include the filename that the error occurred in. This can make tracking down errors tedious.

The cause appears to be how Webpack 5 formats errors combined with how formatWebpackMessages processes those errors. The message object passed to formatMessage in [formatWebpackMessages.js](https://github.com/facebook/create-react-app/blob/main/packages/react-dev-utils/formatWebpackMessages.js) contains separate message and file properties and in the case of TypeScript errors, the file name is not included in the message property, only the file property.

As a workaround for myself I have changed the formatMessage function to include the file property. This is hacky, since for one thing it duplicates the filename in the case that the message property includes the filename, but gets me past having to search for the file manually.

I changed lines 22-23 from

```
  } else if ('message' in message) {
    lines = message['message'].split('\n');
```

to

```
  } else if ('message' in message) {
    lines = message['message'].split('\n');

    if ('file' in message) {
      lines.unshift(message.file);
    }  
```

### Did you try recovering your dependencies?

Not Applicable

### Which terms did you search for in User Guide?

Not Applicable

### Environment

Environment Info:

  current version of create-react-app: 5.0.1
  running from C:\Users\prijks\AppData\Local\npm-cache\_npx\c67e74de0542c87c\node_modules\create-react-app

  System:
    OS: Windows 10 10.0.19042
    CPU: (20) x64 Intel(R) Core(TM) i9-10900X CPU @ 3.70GHz
  Binaries:
    Node: 16.13.0 - C:\Program Files\nodejs\node.EXE
    Yarn: Not Found
    npm: 8.1.0 - C:\Program Files\nodejs\npm.CMD
  Browsers:
    Chrome: Not Found
    Edge: Spartan (44.19041.1266.0), Chromium (101.0.1210.32)
    Internet Explorer: 11.0.19041.1566
  npmPackages:
    react: ^18.1.0 => 18.1.0
    react-dom: ^18.1.0 => 18.1.0
    react-scripts: 5.0.1 => 5.0.1
  npmGlobalPackages:
    create-react-app: Not Found

### Steps to reproduce

1. Create a new app with `npx create-react-app format-message-test --template cra-template-typescript`
2. Introduce a TypeScript error to the app (e.g. by adding an invalid import to App.tsx) - note that it needs a TypeScript error and not a syntax error
3. Compile with `npm run build`

### Expected behavior

Compile error includes the filename 

### Actual behavior

Compile error does not include the filename. The output I get when running the steps above is:

```
> react-scripts build

Creating an optimized production build...
Failed to compile.

TS2307: Cannot find module 'not-a-real-thing' or its corresponding type declarations.
    1 | import React from 'react';
    2 | import logo from './logo.svg';
  > 3 | import foobar from 'not-a-real-thing';
      |                    ^^^^^^^^^^^^^^^^^^
    4 | import './App.css';
    5 |
    6 | function App() {
```

Note there is no filename in that output.

### Reproducible demo

[Demo](https://github.com/prijks/react-webpack-message-test)

Clone the demo, then install dependencies, then run the build script.

Can also be reproduced by creating a project following the reproduce steps above
