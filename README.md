# Abode Cam 2 to RTC

> **Fork notice:** this is a fork of [tradel/hassio-addons](https://github.com/tradel/hassio-addons).
> The Abode Camera Streaming add-on was created by Todd Radel ([@tradel](https://github.com/tradel)),
> and all credit for it goes to him. This fork adds one fix: the add-on reuses a single Abode
> session instead of signing in on every stream, which stopped a flood of sign-in emails and
> account rate-limiting. The fix is offered upstream in
> [tradel/hassio-addons#17](https://github.com/tradel/hassio-addons/pull/17).

## Add-ons

This repository contains the following add-ons:

### [Abode Camera Streaming Support](./abode2rtc)

![Supports aarch64 Architecture][aarch64-shield]
![Supports amd64 Architecture][amd64-shield]
![Does not support armhf Architecture][armhf-shield]
![Supports armv7 Architecture][armv7-shield]
![Supports i386 Architecture][i386-shield]

_Live video from Abode cameras, including the Abode Cam 2, over one persistent Abode session with no sign-in email per stream._

[![Open your Home Assistant instance and show the add-on.](https://my.home-assistant.io/badges/supervisor_addon.svg)](https://my.home-assistant.io/redirect/supervisor_addon/?addon=82aaea8b_abode2rtc&repository_url=https%3A%2F%2Fgithub.com%2Fduvallg%2Fha-abode-cam2)

<!--

Notes to developers after forking or using the github template feature:
- While developing comment out the 'image' key from 'example/config.yaml' to make the supervisor build the addon
  - Remember to put this back when pushing up your changes.
- When you merge to the 'main' branch of your repository a new build will be triggered.
  - Make sure you adjust the 'version' key in 'example/config.yaml' when you do that.
  - Make sure you update 'example/CHANGELOG.md' when you do that.
  - The first time this runs you might need to adjust the image configuration on github container registry to make it public
  - You may also need to adjust the github Actions configuration (Settings > Actions > General > Workflow > Read & Write)
- Adjust the 'image' key in 'example/config.yaml' so it points to your username instead of 'home-assistant'.
  - This is where the build images will be published to.
- Rename the example directory.
  - The 'slug' key in 'example/config.yaml' should match the directory name.
- Adjust all keys/url's that points to 'home-assistant' to now point to your user/fork.
- Share your repository on the forums https://community.home-assistant.io/c/projects/9
- Do awesome stuff!
 -->

[aarch64-shield]: https://img.shields.io/badge/aarch64-yes-green.svg
[amd64-shield]: https://img.shields.io/badge/amd64-yes-green.svg
[armhf-shield]: https://img.shields.io/badge/armhf-no-red.svg
[armv7-shield]: https://img.shields.io/badge/armv7-yes-green.svg
[i386-shield]: https://img.shields.io/badge/i386-yes-green.svg
