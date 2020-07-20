---
title: "Security Recommendations For Github Actions"
date: 2020-07-20T12:50:44-05:00
draft: false
categories: [
  "security"
]
tags: [
    "security",
]
---

I recently moved our continuous integration (CI) tests from Travis-CI to Github Actions for [Brave's](http://brave.com/) [Publisher application](https://github.com/brave-intl/publishers). When [working on the migration](https://github.com/brave-intl/publishers/pull/2784) I found there was a lot of 3rd party actions that you can integrate into your workflows. However that raises the question: Are they secure?

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Are there good security recommendations for using <a href="https://twitter.com/github?ref_src=twsrc%5Etfw">@github</a> actions safely?<br><br>Beyond &quot;audit 3rd party actions and pin them by commit hash&quot;, what else should I be concerned about?</p>&mdash; Julien Vehent (@jvehent) <a href="https://twitter.com/jvehent/status/1285246296702562305?ref_src=twsrc%5Etfw">July 20, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>


I begin to think about this a little more and with the help of some of my coworkers came up with the following notes.

### Reduce use of 3rd-party actions

This is the same concept as with dependency management. Most of what can be done in Github Actions can be done simply by using the normal actions workflow and bash. Take this step for example, we don't have to use a 3rd party bundle install action, instead we can run the commands to execute "Bundle Install"

```yml
- name: Bundle install
  run: |
    bundle config path vendor/bundle
    bundle install --jobs 4 --retry 3
```


### Always use commit hashes for the Github Actions

Action versions [can be unsafe](https://julienrenaux.fr/2019/12/20/github-actions-security-risk/), as the maintainer or attacker can later revise the version.

### Do not use hosted runners for public repos

This one might be a little of a stretch, but unless you can guarantee the runner is ephemeral and virtualised then code secrets could be leaked to other processes. For extra-security you can [host your own Github Runner](https://docs.github.com/en/actions/hosting-your-own-runners/about-self-hosted-runners). However be careful, someone may be able to submit a pull request against the project and someone could abuse that functionality.

### Don't use secrets that could compromise your application

Keep production values out of your Github account, and try to use dummy values if you need to insert secrets into the code. This might not always work for every codebase so consider regularly rotating secrets throughout the year to prevent leakage.

### For iOS, use fastlane match for credentials storage

This advice was presented by a coworker of mine. His advice was to use Fastline for cerdentials storage because the repo will be completely encrypted with a password known only to the builder (a secret).

### Consider integrating tooling

Adding tooling around your code and secrets greatly reduces your risk for expore. Amazon's [Git Secrets](https://github.com/awslabs/git-secrets) hook or [Mozilla's Secrets OPerationS (SOPS)](https://github.com/mozilla/sops) are great options to integrate.
