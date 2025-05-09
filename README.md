# Heroku buildpack: swift

This is a Heroku buildpack for Swift apps that are powered by the Swift Package Manager.

## Usage

Example usage:

```shell
$ ls
Procfile Package.swift Sources

$ heroku create --buildpack vapor/vapor

$ git push heroku main
-----> Building on the Heroku-24 stack
-----> Using buildpack: vapor/vapor
-----> Swift app detected
-----> No PACKAGE_DIR config variable is set. Using the repo root.
-----> Using latest Swift version
-----> Installing swiftly...
...
-----> Installing Swift...
...
-----> Building package (release configuration, static stdlib)...
...
```

You can also add it to upcoming builds of an existing application:

```shell
$ heroku buildpacks:set vapor/vapor
```

The buildpack will detect your app as Swift if it has a `Package.swift` file in the
repository. You can customize the path for the Swift package to deploy by setting the
`PACKAGE_DIR` config setting.

### Procfile

Using the Procfile, you can set the process to run for your web server. Any
binaries built from your Swift source using swift package manager will
be placed in your $PATH.

Example Procfile for Vapor 4 apps:

```
web: App serve --env production --hostname 0.0.0.0 --port $PORT
```

### Specify a Swift version

The buildpack defaults to the latest release version of Swift, but can be configured to install older versions or development snapshots.

If you want to use a specific version of the Swift toolchain, you can pin its version by creating a file called `.swift-version`
in the root of the project folder (the repository root, unless configured with the `PACKAGE_DIR` setting),
or by setting a `SWIFT_VERSION` configuration variable on Heroku, then deploying again.

```shell
$ echo '5.10.1' > .swift-version
$ git add .swift-version
$ git commit -m "Pin Swift version to 5.10.1"
$ git push heroku main
```

Or:

```shell
$ heroku config:set SWIFT_VERSION=5.10.1
$ git commit -m "Pin Swift version to 5.10.1" --allow-empty
$ git push heroku main
```

### Active build configuration

By default, the buildpack will use the `release` build configuration to enable compiler optimizations. If you are experiencing mysterious crashes, you can try disabling them by setting the `SWIFT_BUILD_CONFIGURATION` config variable to `debug`, then redeploying.

```shell
$ heroku config:set SWIFT_BUILD_CONFIGURATION=debug
$ git commit -m "Change to debug configuration on Heroku" --allow-empty
$ git push heroku master
...
remote: -----> Building package (debug configuration, static stdlib)
...
```

### Statically linked standard library

Swift supports static linkage for its standard library and Foundation, which has two advantages:

1. The resulting binary is more self-contained, and can run on other machines running the same version of the same distribution _without having to install the Swift toolchain._
1. As only the relevant parts of the standard library/Foundation are copied into the binary, the resulting image will be smaller. A smaller image deploys faster too.

The buildpack uses static linkage by default.

However, if your project does not build, throwing errors about missing symbols for example, you can opt-in for the previous, dynamic linkage.

Define a configuration variable called `SWIFT_DYNAMIC_STDLIB` with any non-empty value, then deploy again.

```shell
$ heroku config:set SWIFT_DYNAMIC_STDLIB=true
$ git commit -m "Change to dynamic stdlib on Heroku" --allow-empty
$ git push heroku master
```

To use static linkage again, _unset_ the configuration variable and deploy again.

```shell
$ heroku config:unset SWIFT_DYNAMIC_STDLIB
$ git commit -m "Change to dynamic stdlib on Heroku" --allow-empty
$ git push heroku master
```

### Other build arguments

If you want to pass extra flags to the `swift build` command, you can do so by setting the `SWIFT_BUILD_FLAGS` config variable.

### Hooks

You can place custom scripts to be run before and after compiling and installing your Swift
source code inside the following files in your repository:

- `bin/pre_compile`
- `bin/post_compile`
- `bin/pre_install`
- `bin/post_install`

This is useful if you need to customize the final image.

The buildpack passes three arguments to the scripts: the BUILD_DIR, CACHE_DIR and ENV_DIR paths that Heroku provided for the build. See the [Buildpack API documentation](https://devcenter.heroku.com/articles/buildpack-api) for details.

#### Example: using private dependencies

For larger projects with private dependencies, using [heroku-buildpack-github-netrc](https://elements.heroku.com/buildpacks/heroku/heroku-buildpack-github-netrc) is a solid solution – as long as the dependencies are on GitHub.

The same idea – using .netrc to pass in credentials – works for GitLab and other providers just as well.

The following `pre_compile` script creates a .netrc file from configuration variables.
Save the script as `bin/pre_compile` in the root of your project.

    # Load private git credentials from the app configuration
    GIT_PRIVATE_DOMAIN=`cat $ENV_DIR/GIT_PRIVATE_DOMAIN | tr -d '[[:space:]]'`
    GIT_PRIVATE_USER=`cat $ENV_DIR/GIT_PRIVATE_USER | tr -d '[[:space:]]'`
    GIT_PRIVATE_PASSWORD=`cat $ENV_DIR/GIT_PRIVATE_PASSWORD | tr -d '[[:space:]]'`

    # Create .netrc file with credentials
    echo "machine $GIT_PRIVATE_DOMAIN" >> "$HOME/.netrc"
    echo "login $GIT_PRIVATE_USER" >> "$HOME/.netrc"
    echo "password $GIT_PRIVATE_PASSWORD" >> "$HOME/.netrc"

    # Create cleanup script so the dyno does not see these values at runtime
    mkdir -p "$BUILD_DIR/.profile.d"
    echo "unset GIT_PRIVATE_DOMAIN" >> "$BUILD_DIR/.profile.d/netrc.sh"
    echo "unset GIT_PRIVATE_USER" >> "$BUILD_DIR/.profile.d/netrc.sh"
    echo "unset GIT_PRIVATE_PASSWORD" >> "$BUILD_DIR/.profile.d/netrc.sh"

Then define the GIT_PRIVATE_DOMAIN, GIT_PRIVATE_USER and GIT_PRIVATE_PASSWORD configuration variables on the Heroku dashboard,
or via the `heroku config:set` command.

See the following example for the latter:

    heroku config:set GIT_PRIVATE_DOMAIN=gitlab.com \
      GIT_PRIVATE_USER=user@organization.com \
      GIT_PRIVATE_PASSWORD=As1D2f34

Then commit and deploy the project again.

## Using the latest source code

The `vapor/vapor` buildpack from the [Heroku Buildpack Registry](https://devcenter.heroku.com/articles/buildpack-registry) represents the latest stable version of the buildpack. If you'd like to use the source code from this Github repository, you can set your buildpack to the Github URL:

```shell
$ heroku buildpacks:set https://github.com/vapor-community/heroku-buildpack.git
```
