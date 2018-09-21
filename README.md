fastlane documentation
================
# Installation

Make sure you have the latest version of the Xcode command line tools installed:

```
xcode-select --install
```

Install _fastlane_ using
```
[sudo] gem install fastlane -NV
```
or alternatively using `brew cask install fastlane`

# Available Actions
### register_new_devices
```
fastlane register_new_devices
```
Update the list of devices for existing provision profile
### install_deps
```
fastlane install_deps
```
Install all dependencies
### to_appstore
```
fastlane to_appstore
```
Add the build to App Store For verification
### to_testflight
```
fastlane to_testflight
```
Add new build to TestFlight
### ship_to_fabric
```
fastlane ship_to_fabric
```
Add new build to Fabric, with possible option: `flavor: staging`

----

This README.md is auto-generated and will be re-generated every time [fastlane](https://fastlane.tools) is run.
More information about fastlane can be found on [fastlane.tools](https://fastlane.tools).
The documentation of fastlane can be found on [docs.fastlane.tools](https://docs.fastlane.tools).
