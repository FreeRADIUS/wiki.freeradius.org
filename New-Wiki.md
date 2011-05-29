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
5. Convert markup to [[reStructuredText|http://docutils.sourceforge.net/docs/ref/rst/restructuredtext.htm] or MarkDown
6. Save the page
7. Get warm fuzzy glow from contributing to open source

Also please report any bugs here:
[[https://github.com/github/gollum/issues?_pjax=true&state=open]]

## Known Issues
* Markdown pages don't display TOC, this is a upstream bug in the 1.3.0 version of gollum
* No delete or rename functionality, this is a upstream bug in the 1.3.0 version of gollum
* When creating pages, you are only prompted to login after attempting to submit modifications; this is somewhat of a design flaw in gollum, not specifying a get url for creating pages. A workaround will be written and implemented soon.

## Pages that need converting as a priority (please mark here when converted)

### Pages to convert to markdown
* http://wiki.freeradius.org/Build
* http://wiki.freeradius.org/Modules1
* http://wiki.freeradius.org/RADIUS-Clients
* http://wiki.freeradius.org/Cisco
* http://wiki.freeradius.org/PopTop
* http://wiki.freeradius.org/Attribute%20support%20by%20processing%20list
* http://wiki.freeradius.org/Attrs

### Pages to convert to RST
* http://wiki.freeradius.org/Modules2

## Todo
* Add google as omniauth provider - Arran CB
* Implement proper fix for page creation - Arran CB
* Fix TOC for markdown pages - Arran CB
* Set up anonymous git access for wiki - Alan D
* Set up commit log to user mailing list - Alan D