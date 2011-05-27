One of the largest complaints with FreeRADIUS is the lack of comprehensive documentation.

The previous wiki  has served its purpose, but has ultimately failed to provide an up-to-date, well organised source of documentation.

The current major problems with the wiki are:

* spam users - which meant we had to lock registration, and discouraged new users from contributing
* exporting information - all pages are stored in an sql lite instance, which makes it hard to automatically roll pages into releases
* formatting information - Information stored in the wiki is in the Media wiki format, whereas the documentation bundled with FreeRADIUS is either unformatted or in rst format.

To try and solve these issues and glue everything together a bit more, we've been working to set up a new wiki based on Gollum. Gollum is a ruby on rails application which exposes a git repository as wiki site. Gollum can render files in many markup languages including plaintext, RST and Mediawiki format, which means we can import all current server documentation, all current wiki documentation and have them neatly presented in a single wiki site. Neat huh?

But what about spam and registration? Well by default gollum doesn't authenticate anyone. But because it's a rails application we can drop in a library called 'OmniAuth' which uses Oauth to authenticate a bunch of providers.

This allows us to leverage authentication and spam account prevention services of providers like GitHub, Facebook and Twitter.

Unfortunately the new wiki needs some fixes. The mediawiki page format renderer in gollum isn't perfect, so we need to convert the pages which don't render correctly to RST as a priority.

If you want to help out, please do the following:

1. Sign up for Facebook, Twitter or GitHub
2. Pick a page where the MediaWiki format doesn't render correctly
3. Click the 'Edit page' button
4. Change edit mode from MediaWiki to reStructuredText
5. Convert markup to [[reStructuredText|http://docutils.sourceforge.net/docs/ref/rst/restructuredtext.htm]
6. Save the page
7. Get warm fuzzy glow from contributing to open source

Also please report any bugs here:
[[https://github.com/github/gollum/issues?_pjax=true&state=open]]