---
title: Use Sauce Labs with Travis CI
layout: en

---

Travis CI integrates with [Sauce Labs](https://saucelabs.com), a browser and
mobile testing platform. It integrates well with Selenium, for instance.

The integration automatically sets up a tunnel required to get started testing
with it. For that purpose, it uses Sauce Connect.

Note that due to security restrictions, the Sauce Labs addon is not available on pull
request builds unless you use the [JWT Addon](/user/jwt).

## Setup Sauce Connect

[Sauce Connect][sauce-connect] securely proxies browser traffic between Sauce
Labs' cloud-based VMs and your local servers. Sauce Connect uses ports 443 and 80 for
communication with the Sauce Labs cloud. If you're using Sauce Labs for your Selenium
tests, this makes connecting to your web server a lot easier.

[sauce-connect]: https://docs.saucelabs.com/secure-connections/sauce-connect/

First, [sign up][sauce-sign-up] with Sauce Labs if you haven't already (it's
[free][open-sauce] for Open Source projects), and get your access key from your
[account page][sauce-account]. Once you have that, add this to your .travis.yml
file:

```yaml
addons:
  sauce_connect:
    username: "Your Sauce Labs username"
    access_key: "Your Sauce Labs access key"
```
{: data-file=".travis.yml"}

[sauce-sign-up]: https://signup.saucelabs.com/signup/trial

[sauce-account]: https://saucelabs.com/account

[open-sauce]: https://saucelabs.com/open-source

If you don't want your access key publicly available in your repository, you
can encrypt it with `travis encrypt "your-access-key"` (see [Encryption Keys][encryption-keys]
for more information on encryption), and add the pull request safe secure (See [JWT Addon](/user/jwt).
string as such:

```yaml
addons:
  sauce_connect:
    username: "Your Sauce Labs username"
  jwt:
    secure: "The secure string output by `travis encrypt SAUCE_ACCESS_KEY=Your Sauce Labs access key`"
```
{: data-file=".travis.yml"}

You can also add the `username` and `access_key` as environment variables if you
name them `SAUCE_USERNAME` and `SAUCE_ACCESS_KEY`, respectively. In that case,
all you need to add to your .travis.yml file is this:

```yaml
addons:
  sauce_connect: true
```
{: data-file=".travis.yml"}

[encryption-keys]: /user/encryption-keys/

[jwt]: /user/jwt/

To allow multiple tunnels to be open simultaneously, Travis CI opens a
Sauce Connect [Tunnel Pool][identified-tunnels]. Make sure you are sending
the `TRAVIS_JOB_NUMBER` environment variable when you are opening the connection
to Sauce Labs' selenium grid, as the desired `tunnel-name` capability,
or it will not be able to connect to the server running on the VM.

[identified-tunnels]: https://docs.saucelabs.com/secure-connections/sauce-connect/setup-configuration/high-availability/#tunnel-pools

How this looks will depend on the client library you're using, in
Ruby's [selenium-webdriver][ruby-bindings] bindings:

```
caps = Selenium::WebDriver::Remote::Capabilities.firefox({
  'tunnel-name' => ENV['TRAVIS_JOB_NUMBER']
})
driver = Selenium::WebDriver.for(:remote, {
  url: 'http://username:access_key@ondemand.saucelabs.com/wd/hub',
  desired_capabilities: caps
})
```

[ruby-bindings]: https://code.google.com/p/selenium/wiki/RubyBindings

## Additional Options

Sometimes, you may need to pass additional options to Sauce Connect. Currently
supported parameters are

- `direct_domains`
- `no_ssl_bump_domains`
- `tunnel_domains`

As an example, you may need `--direct-domains` option in case [some HTTPS domains
fail to work with Sauce Connect](https://support.saucelabs.com/hc/en-us/articles/225267468--Bad-Gateway-or-Security-Warnings-When-Using-Sauce-Connect-for-Testing-Web-Applications-over-HTTPS):

```yaml
addons:
  sauce_connect:
    username: "Your Sauce Labs username"
    direct_domains: example.org,*.foobar.com
  jwt:
    secure: "The secure string output by `travis encrypt SAUCE_ACCESS_KEY=Your Sauce Labs access key`"
```
{: data-file=".travis.yml"}

## Build Config Reference

You can find more information on the build config format for [Sauce Connect](https://config.travis-ci.com/ref/job/addons/sauce_connect) in our [Travis CI Build Config Reference](https://config.travis-ci.com/).
