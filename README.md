# Release

This is a Github action that works with [autotagger](https://github.com/butlerlogic/action-autotag) and the [cross-runtime template](https://github.com/author/template-cross-runtime) to build and release JavaScript libraries for different runtimes.

The [cross-runtime template](https://github.com/author/template-cross-runtime) will build multiple versions of a library for Node.js and browsers.

For example, the [@author.io/shell](https://github.com/author/shell) library runs in both Node.js and browsers, but releases code under these packages:

- **browser-shell** (ES Module edition for modern browsers)
- **browser-shell-debug** (ES Module debugging edition for modern browsers)
- **browser-shell-es6** (ES6 edition for legacy browsers)
- **browser-shell-es6-debug** (ES6 debugging edition for legacy browsers)
- **node-shell** (ES Module edition for Node 12+)
- **node-shell-debug** (ES Module debugging edition for Node 12+)
- **node-shell-legacy** (CommonJS edition for legacy Node)
- **node-shell-legacy-debug** (CommonJS debugging edition for legacy Node)

That's alot of modules, all generated from the same source code. This action automates the process of releasing them all.

## Usage

This action is meant to be run after a new tag is created, where the tag represents a new version to release. The [autotagger](https://github.com/butlerlogic/action-autotag) action is best suited for this.

Once a new tag is created, this action can build a release containing all of the modules.

The following is an example `.github/main.workflow` that will execute when a new tag is created.

```yaml
name: My Workflow

on:
  push:
    tags:
      - '*'

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - uses: author/action-release@stable
      with:
        GITHUB_TOKEN: "${{ secrets.GITHUB_TOKEN }}"
        before: "npm run build"
        scan:
          - "./dist"
```

To make this work, the workflow must have the checkout action _before_ the release action.

This **order** is important!

```yaml
- uses: actions/checkout@master
- uses: butlerlogic/action-autotag@1.0.0
```

> If the repository is not checked out first, the autotagger cannot find the package.json file.

## Configuration

The `GITHUB_TOKEN` must be passed in. Without this, it is not possible to create a new tag. Make sure the autotag action looks like the following example:

```yaml
- uses: butlerlogic/action-autotag@1.0.0
  with:
    GITHUB_TOKEN: "${{ secrets.GITHUB_TOKEN }}"
```

The action will automatically extract the token at runtime. **DO NOT MANUALLY ENTER YOUR TOKEN.** If you put the actual token in your workflow file, you'll make it accessible (in plaintext) to anyone who ever views the repository (it will be in your git history).

### Optional Configurations

There are several options to customize how the tag is created.

1. `package_root`

    By default, autotag will look for the `package.json` file in the project root. If the file is located in a subdirectory, this option can be used to point to the correct file.

    ```yaml
    - uses: butlerlogic/action-autotag@1.0.0
      with:
        GITHUB_TOKEN: "${{ secrets.GITHUB_TOKEN }}"
        package_root: "/path/to/subdirectory"
    ```

1. `tag_prefix`

    By default, `package.json` uses [semantic versioning](https://semver.org/), such as `1.0.0`. A prefix can be used to add text before the tag name. For example, if `tag_prefx` is set to `v`, then the tag would be labeled as `v1.0.0`.

    ```yaml
    - uses: butlerlogic/action-autotag@1.0.0
      with:
        GITHUB_TOKEN: "${{ secrets.GITHUB_TOKEN }}"
        tag_prefix: "v"
    ```

1. `tag_suffix`

    Text can also be applied to the end of the tag by setting `tag_suffix`. For example, if `tag_suffix` is ` (beta)`, the tag would be `1.0.0 (beta)`. Please note this example violates semantic versioning and is merely here to illustrate how to add text to the end of a tag name if you _really_ want to.

    ```yaml
    - uses: butlerlogic/action-autotag@1.0.0
      with:
        GITHUB_TOKEN: "${{ secrets.GITHUB_TOKEN }}"
        tag_suffix: " (beta)"
    ```

1. `tag_message`

    This is the annotated commit message associated with the tag. By default, a
    changelog will be generated from the commits between the latest tag and the new tag (HEAD). Setting this option will override it witha custom message.

    ```yaml
    - uses: butlerlogic/action-autotag@1.0.0
      with:
        GITHUB_TOKEN: "${{ secrets.GITHUB_TOKEN }}"
        tag_message: "Custom message goes here."
    ```

1. `version`

    Explicitly set the version instead of automatically detecting from `package.json`.
    Useful for non-JavaScript projects where version may be output by a previous action.

    ```yaml
    - uses: butlerlogic/action-autotag@1.0.0
      with:
        GITHUB_TOKEN: "${{ secrets.GITHUB_TOKEN }}"
        version: "${{ steps.previous_step.outputs.version }}"
    ```

## Developer Notes

If you are building an action that runs after this one, be aware this action produces several [outputs](https://help.github.com/en/articles/metadata-syntax-for-github-actions#outputs):

1. `tagname` will be empty if no tag was created, or it will be the value of the new tag.
1. `tagsha`: The SHA of the new tag.
1. `taguri`: The URI/URL of the new tag reference.
1. `tagmessage`: The messge applied to the tag reference (this is what shows up on the tag screen on Github).
1. `version` will be the version attribute found in the `package.json` file.

---

## Credits

This action was written and is primarily maintained by [Corey Butler](https://github.com/coreybutler).

# Our Ask...

If you use this or find value in it, please consider contributing in one or more of the following ways:

1. Click the "Sponsor" button at the top of the page.
1. Star it!
1. [Tweet about it!](https://twitter.com/intent/tweet?hashtags=github,actions&original_referer=http%3A%2F%2F127.0.0.1%3A91%2F&text=I%20am%20automating%20my%20workflow%20with%20the%20Autotagger%20Github%20action!&tw_p=tweetbutton&url=https%3A%2F%2Fgithub.com%2Fmarketplace%2Factions%2Fautotagger&via=goldglovecb)
1. Fix an issue.
1. Add a feature (post a proposal in an issue first!).

Copyright &copy; 2019 ButlerLogic, Corey Butler, and Contributors.
