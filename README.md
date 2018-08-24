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
remote: -----> Installing clang 5.0.0
remote: -----> Installing swiftenv
remote: -----> Installing Swift 4.1.1
remote: -----> Building package (release configuration)
remote: -----> Installing dynamic libraries
remote: -----> Installing binaries
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

Example Procfile for Vapor 2 apps:

```
web: Run --env=production --port=$PORT
```

Example Procfile for Vapor 3 apps (Swift 4.1 required):

```
web: Run serve --env production --hostname 0.0.0.0 --port $PORT
```

### Specify a Swift version

You can also customise the version of Swift used with a `.swift-version` file
in your repository:

```shell
$ cat .swift-version
4.1.1
```

The `.swift-version` file is completely compatible with
[swiftenv](http://github.com/kylef/swiftenv).

**NOTE**: *Since there are frequent Swift language changes, it's advised that
you pin to your Swift version.*

### Active build configuration

By default, the buildpack will use the `release` build configuration to enable compiler optimizations. If you are experiencing mysterious crashes, you can try disabling them by setting the `SWIFT_BUILD_CONFIGURATION` to `debug`, then redeploying.

```shell
$ heroku config:set SWIFT_BUILD_CONFIGURATION=debug
$ git commit -m "Change to debug configuration on Heroku" --allow-empty
$ git push heroku master
...
remote: -----> Building package (debug configuration)
...
```

### Hooks

You can place custom scripts to be ran before and after compiling your Swift
source code inside the following files in your repository:

- `bin/pre_compile`
- `bin/post_compile`

This is useful if you would need to install any other dependencies.

## Using the latest source code

The `vapor/vapor` buildpack from the [Heroku Buildpack Registry](https://devcenter.heroku.com/articles/buildpack-registry) represents the latest stable version of the buildpack. If you'd like to use the source code from this Github repository, you can set your buildpack to the Github URL:

```sh-session
$ heroku buildpacks:set https://github.com/vapor-community/heroku-buildpack.git
```
