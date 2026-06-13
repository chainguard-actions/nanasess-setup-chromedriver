# setup-chromedriver

[![Test chromedriver on *NIX](https://github.com/nanasess/setup-chromedriver/actions/workflows/test.yml/badge.svg)](https://github.com/nanasess/setup-chromedriver/actions/workflows/test.yml)
[![Test chromedriver on Windows](https://github.com/nanasess/setup-chromedriver/actions/workflows/windows.yml/badge.svg)](https://github.com/nanasess/setup-chromedriver/actions/workflows/windows.yml)
[![GitHub](https://img.shields.io/github/license/nanasess/setup-chromedriver)](./LICENSE)
[![GitHub Sponsors](https://img.shields.io/github/sponsors/nanasess)](https://github.com/sponsors/nanasess)

This action sets up a [ChromeDriver](https://chromedriver.chromium.org/) for use in actions

## OS/Platform support

- ubuntu-latest, ubuntu-22.04 and ubuntu-20.04
- macos-latest, macos-12 and macos-11
- windows-latest, windows-2022 and windows-2019

# Usage

See [action.yml](action.yml)

## for ubuntu-latest, macos-latest

``` yaml
steps:
- uses: actions/checkout@v2
- uses: nanasess/setup-chromedriver@v2
  with:
    # Optional: do not specify to match Chrome's version
    chromedriver-version: '88.0.4324.96'
- run: |
    export DISPLAY=:99
    chromedriver --url-base=/wd/hub &
    sudo Xvfb -ac :99 -screen 0 1280x1024x24 > /dev/null 2>&1 & # optional
 ```

## for windows-latest

``` yaml
steps:
- uses: actions/checkout@v2
- uses: nanasess/setup-chromedriver@v2
  with:
    # Optional: do not specify to match Chrome's version
    chromedriver-version: '88.0.4324.96'
- run: chromedriver --url-base=/wd/hub &
 ```

## Privacy

This Action contacts Chainguard's licensing server to verify authorization. Connection metadata (IP address, GitHub repository identifier, timestamp, and any metadata encoded in the auth token) is transmitted to Chainguard, Inc. even if authorization is denied in accordance with our [Privacy Notice](https://www.chainguard.dev/legal/privacy-notice)
