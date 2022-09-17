---
layout: default
permalink: /globus-examples/
title: Globus Examples
search_exclude: false
---

# Globus Examples

## Premise and Intentions

This was also posted an [issue on the Globus CLI
repo](https://github.com/globus/globus-cli/issues/679) to get feedback
from the Globus developers. This goes back to a conversation that
happened while I was part of the Globus team when we created the
[automation examples](https://github.com/globus/automation-examples)
repository, about how to handle community contributions.

My intention is to post a few (maybe several) end-to-end examples of
how to do things for various use cases--when everything is mostly
working. I.e., useful demo, teaching, and sys admin script
references. These are coming from my work with projects at UCSD, and
sharing them publicly is the best way for folks on campus to find them.

The examples largely build on the Globus Python SDK and use the
command line so that it's easy for someone to both read and run
them. Which is why I reached out to the Globus team, because I'm
certain there's a ridiculously long wish list for the Globus SDK and
CLI that include some of the things I'll be covering. And I fully appreciate
the difference between me putting out script with poor error
handling that avoids edge cases compared to what the supported
releases need to achieve.

With that framing, here are my intentions as I put these out in the
world.

### Vulnerabilities

If you see something egregiously wrong, particularly regarding the
handling of tokens, permissions, roles, etc., please let me know. I
don't expect anyone to vet or review these examples, but I will take
seriously any reports of a potential risk to someone using them.

### Deprecating Examples

When the CLI or some other supported tool from Globus add the same
functionality, I'll update my examples to reference it, or even remove
the example. This is the part about the SDK and CLI roadmap. I know
some of these things will be coming; for now, plese treat the examples
as a way for the Globus community to adopt Globus product features as
those features go from the API, to the SDK, to the CLI.

### Please Reuse or Take Inspiration

If anything in an example looks useful, please feel free to reuse it,
I would be honored. Realistically, you're not going to find the actual
code useful, but maybe a feature will be worth implementing. For
example, I think the abiltity to [pass the client ID and secret via a
file](https://rpwagner.github.io/notes/2022/09/16/globus-https-put.html#confidential-client)
is a good addition to [using environment
variables](https://docs.globus.org/cli/environment_variables/#client_credentials_with_globus_cli_client_id). I've
started looking at the Globus CLI code to see how to add the option
through a pull request, so we'll see if I can [contribute
directly](https://github.com/globus/globus-cli/blob/main/CONTRIBUTING.adoc).

Likewise, I'd be glad to see these included in training materials,
blog posts, or other sharing if it's worthwhile from your
perspective. However, to my comment about about where to host these, I
chose not to do a pull request to the automation examples repo or
elsewhere. I think it's better to see if how the Globus user community
responds to the examples and if it results in a Globus- or
community-managed space for contributions.

## Thanks

Finally, thanks to the Globus on everything that makes these possible.

## List of Examples

- HTTPS PUT [Description](https://rpwagner.github.io/notes/2022/09/16/globus-https-put.html), [Script](https://github.com/rpwagner/serverless-data/blob/main/bin/globuscollectionput.py)


