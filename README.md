# Heroku buildpack: swift

This is a Heroku buildpack for Swift apps that are powered by the Swift Package Manager.

## Usage

Example usage:

```shell
$ ls
Procfile Package.swift Sources

$ heroku create --buildpack vapor/vapor

$ git push heroku master
remote: -----> Swift app detected
remote: -----> Using Swift 5.5.1 (default)
remote: -----> Using built-in clang (Swift 5.5.1)
remote: -----> Installing swiftenv
remote: -----> Installing Swift 5.5.1
...
```

You can also add it to upcoming builds of an existing application:

```shell
$ heroku buildpacks:set vapor/vapor
```

The buildpack will detect your app as Swift if it has a `Package.swift` file in
the root.

### Procfile

Using the Procfile, you can set the process to run for your web server. Any
binaries built from your Swift source using swift package manager will
be placed in your $PATH.

Example Procfile for Vapor 3 and 4 apps:

```
web: Run serve --env production --hostname 0.0.0.0 --port $PORT
```

Example Procfile for Vapor 2 apps:

```
web: Run --env=production --port=$PORT
```

### Specify a Swift version

The buildpack defaults to Swift 5.5.1 and will be updated when new Swift versions are released.

If you need to use a specific version of the Swift toolchain, including older versions – for example Swift 4.2.x to retain compatibility with Swift 3 projects, you can pin that version number using a file called `.swift-version` in the root of the project folder, or by setting a `SWIFT_VERSION` configuration variable on Heroku, then deploying again. 

```shell
$ echo '5.4.3' > .swift-version
$ git add .swift-version
$ git commit -m "Pin Swift version to 5.4.3"
$ git push heroku master
```

Or:

```shell
$ heroku config:set SWIFT_VERSION=5.4.3
$ git commit -m "Pin Swift version to 5.4.3" --allow-empty
$ git push heroku master
```

The version format used file is compatible with [swiftenv](http://github.com/kylef/swiftenv).

### Active build configuration

By default, the buildpack will use the `release` build configuration to enable compiler optimizations. If you are experiencing mysterious crashes, you can try disabling them by setting the `SWIFT_BUILD_CONFIGURATION` config variable to `debug`, then redeploying.

```shell
$ heroku config:set SWIFT_BUILD_CONFIGURATION=debug
$ git commit -m "Change to debug configuration on Heroku" --allow-empty
$ git push heroku master
...
remote: -----> Building package (debug configuration)
...
```

### Other build arguments

If you want to pass extra flags to the `swift build` command, you can do so by setting the `SWIFT_BUILD_FLAGS` config variable. The most common use of this feature is to enable _test discovery_ for older versions of Swift.

#### Test discovery (Swift 5.3.3 and below)

> Swift 5.4+ runs test discovery by default, making this section finally obsolete.

Previously, projects with a test target needed either a LinuxMain.swift file, or a build flag that enables test discovery, to build successfully on Linux. Lacking them, the build would fail with an error message like below:

    remote: error: missing LinuxMain.swift file in the Tests directory
    remote:  !     Push rejected, failed to compile Swift app.
    remote: 
    remote:  !     Push failed


The easy and low-maintenance solution was passing the `--enable-test-discovery` build flag via Heroku configuration and attempting to deploy again.

The following example demonstrates this:

```shell
$ heroku config:set SWIFT_BUILD_FLAGS="--enable-test-discovery"
$ git commit -m "Enable test discovery on Heroku" --allow-empty
$ git push heroku master
```

> Note that the empty commit is only required if uncommitted files and the previous deployment was successful.

### Hooks

You can place custom scripts to be run before and after compiling your Swift
source code inside the following files in your repository:

- `bin/pre_compile`
- `bin/post_compile`

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
