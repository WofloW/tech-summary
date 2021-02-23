Issues when upgrading Nodejs to 12

# fsevents install error
Add this code to package.json
```
"resolutions": {
  "**/fsevents": "^1.2.9"
}
```

# How to fix ReferenceError: primordials is not defined in node
https://stackoverflow.com/questions/55921442/how-to-fix-referenceerror-primordials-is-not-defined-in-node

Only work when adding the resolutions in the main project package.json.

We encountered the same issue when updating a legacy project depending on gulp@3.9.1 to Node.js 12+.

These fixes enable you to use Node.js 12+ with gulp@3.9.1 by overriding graceful-fs to version ^4.2.4.

## If you are using yarn v1
```
{
  // Your current package.json contents
  "resolutions": {
    "graceful-fs": "^4.2.4"
  }
}
```

## If you are using npm
Using npm-force-resolutions as a preinstall script, you can obtain a similar result as with yarn v1. You need to modify your package.json this way:

```
{
  // Your current package.json
  "scripts": {
    // Your current package.json scripts
    "preinstall": "npx npm-force-resolutions"
  },
  "resolutions": {
    "graceful-fs": "^4.2.4"
  }
}
```

