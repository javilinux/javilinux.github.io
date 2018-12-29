---
layout: post
title: Install Zendesk Apps Tool(ZAT) on Fedora 29
---

Although in the official [documentation](https://develop.zendesk.com/hc/en-us/articles/360001075048) there is no mention to Fedora or any linux distribution, actually is not that difficult to install ZAT on fedora.

You just need to install the necessary dependencies:
```
dnf install rubygems ruby-devel gcc-c++ rubygem-rake zlib-devel nodejs
```

Nodejs is needed for the *zat validate* command.

And finally install the ruby gem with:

```
gem install zendesk_apps_tools
```

Now you are ready to start building Zendesk Apps , a good starting point could be [Build your first Support app - Part 1: Laying the groundwork](https://develop.zendesk.com/hc/en-us/articles/360001074788)

