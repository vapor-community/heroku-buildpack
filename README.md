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
remote: -----> Using Swift 5.2.1 (default)
remote: -----> Using built-in clang (Swift 5.2.1)
remote: -----> Installing swiftenv
remote: -----> Installing Swift 5.2.1
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

The buildpack defaults to Swift 5.2.1 which will be swiftly updated when new Swift versions are released.

If you need to use a specific version of the Swift toolchain, including older versions – for example Swift 4.2.x to retain compatibility with Swift 3 projects, you can pin that version number using a file called `.swift-version` in the root of the project folder, or by setting a `SWIFT_VERSION` configuration variable on Heroku, then deploying again. 

```shell
$ echo '5.1.5' > .swift-version
$ git add .swift-version
$ git commit -m "Pin Swift version to 5.1.5"
$ git push heroku master
```

Or:

```shell
$ heroku config:set SWIFT_VERSION=5.1.5
$ git commit -m "Pin Swift version to 5.1.5" --allow-empty
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

If you want to pass in extra flags to `swift build`, you can do so using the `SWIFT_BUILD_FLAGS` config variable.

For example, the excellent [swift-backtrace](https://github.com/swift-server/swift-backtrace) library needs debug symbols, which can be enabled by passing `-Xswiftc -g` to the build command. Or, if you want to leverage test discovery instead of generating/writing Linux test manifests, the `--enable-test-discovery` flag is really useful.

The following example shows how to set both:

```shell
$ heroku config:set SWIFT_BUILD_FLAGS="--enable-test-discovery -Xswiftc -g"
$ git commit -m "Enable test discovery and debug symbols on Heroku" --allow-empty
$ git push heroku master
```

Note: while the buildpack does not run tests, having a test manifest or enabling test discovery is mandatory at the time of writing.

### Hooks

You can place custom scripts to be ran before and after compiling your Swift
source code inside the following files in your repository:

- `bin/pre_compile`
- `bin/post_compile`

This is useful if you would need to install any other dependencies.

## Using the latest source code

The `vapor/vapor` buildpack from the [Heroku Buildpack Registry](https://devcenter.heroku.com/articles/buildpack-registry) represents the latest stable version of the buildpack. If you'd like to use the source code from this Github repository, you can set your buildpack to the Github URL:

```shell
$ heroku buildpacks:set https://github.com/vapor-community/heroku-buildpack.git
```
