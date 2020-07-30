# Plugins PoC (External PR Demo)

This repo contains a PoC of how multiple plugins in a single repo will be
tested and released with CircleCI.

The plugin packages lie in `packages`.
Develop each plugin just as you would for a normal plugin.

## Android unit tests

For Android, add a `test` directory under `android/src`, and modify
`android/build.gradle` to add test dependencies like JUnit and Mockito.
These tests can be run with Gradle, by running `flutter build apk` in
`example` and then running `./gradlew testDebugUnitTest` under
`example/android`.

## iOS unit tests

(There might be easier ways to do this, but this is what I have
been able to figure out so far...)

For iOS, first run `flutter build ios --no-codesign` in `example`.
Open the example's Xcode workspace, and add your tests by going to the tests
tab and creating a new test target.
Choose **Runner** as the test target, and use `xxx_pluginTests` for the
Product Name.

Then you can create new Test classes under that target.
Put new Test classes under `ios/Tests` in the package directory
(outside of `example`).
Then you can delete the default test class that Xcode created for you.

Modify `ios/xxx.podspec`, and add the following to the end of the outermost block
(not sure if this step is necessary, but looks like it's done for every plugin with
XCTest):

```ruby
  s.test_spec 'Tests' do |test_spec|
    test_spec.source_files = 'Tests/**/*'
  end
```

Then your tests can be run with `xcodebuild test -workspace Runner.xcworkspace -scheme Runner -destination 'platform=iOS Simulator,name=iPhone 11,OS=13.6'` under `example/ios`, with whatever destination that you like.

If any test dependency is needed, add them in `example/ios/Podfile`.
Also add a line in `ios/xxx.podspec` in the `s.test_spec 'Tests'` block: `test_spec.dependency 'SomeDependency', '1.0'`.

## CI/CD

Currently, a new job is run for each of Flutter, Android and iOS for every plugin,
and Flutter has to be downloaded for every job.

I'm working on speeding up the testing, by merging jobs and reusing resources.
