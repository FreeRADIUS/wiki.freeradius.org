## But what was wrong with the old wiki?

One of the largest complaints with FreeRADIUS is the lack of comprehensive documentation.

The previous wiki  has served its purpose, but has ultimately failed to provide an up-to-date, well organised source of documentation.

The major problems with the previous wiki were:

* spam users - which meant we had to lock registration, and discouraged new users from contributing
* exporting information - all pages are stored in an sql lite instance, which made it hard to automatically roll pages into releases
* formatting information - Information stored in the wiki was/is in the Media wiki format, whereas the documentation bundled with FreeRADIUS is either unformatted or in rst format.

To try and solve these issues and glue everything together a bit more, we setup this new wiki based on Gollum. Gollum is a ruby on rails application which exposes a git repository as wiki site. Gollum can render files in many markup languages including plaintext, RST and Mediawiki format, which means we can import all current server documentation, all current wiki documentation and have them neatly presented in a single wiki site. Neat huh?

But what about spam and registration? Well by default gollum doesn't authenticate anyone. But because it's a rails application we can drop in a library called 'OmniAuth' which uses Oauth to authenticate a bunch of providers.

This allows us to leverage authentication and spam account prevention services of providers like GitHub, Facebook and Twitter.

## Helping out
Unfortunately the new wiki needs some fixes. The mediawiki page format renderer in gollum isn't perfect, so we need to convert the pages which don't render correctly to RST as a priority.

If you want to help out, please do the following:

1. Sign up for Facebook, Twitter or GitHub
2. Pick a page where the MediaWiki format doesn't render correctly
3. Click the 'Edit page' button
4. Change edit mode from MediaWiki to reStructuredText or Markdown
5. Convert markup to [reStructuredText](http://docutils.sourceforge.net/docs/user/rst/quickstart.html) or [Markdown](http://daringfireball.net/projects/markdown/syntax#precode) ([Tables](http://www.justatheory.com/computers/markup/markdown-table-rfc.html), [Cheat sheet](http://hw.libsyn.com/p/8/3/3/8339a864bb8faa83/Markdown_Cheat_Sheet.pdf))
6. Save the page
7. Get warm fuzzy glow from contributing to open source

Also please report any bugs here:
[gollum issue tracker](https://github.com/github/gollum/issues?_pjax=true&state=open)

## Known Issues
* The majority of links are now broken, you can help by editing pages and fixing the links. The pages are now in a directory hierarchy and the links need to be updated. Links are in the format ``[[name|path/page]]``. You'll have to search to figure out where the pages now live.

## Pages that need converting as a priority (please mark here when converted)

### Pages to convert to markdown
* http://wiki.freeradius.org/Build - ACB 29/05/11
* http://wiki.freeradius.org/Modules1
* http://wiki.freeradius.org/RADIUS-Clients
* http://wiki.freeradius.org/Cisco - JC 01/06/11
* http://wiki.freeradius.org/PopTop
* http://wiki.freeradius.org/Attribute%20support%20by%20processing%20list
* http://wiki.freeradius.org/Attrs - JC 01/06/11

### Pages to convert to RST
* http://wiki.freeradius.org/Modules2 - JC 31/05/11
* http://wiki.freeradius.org/Operators - ACB 01/06/11
* http://wiki.freeradius.org/FreeRADIUS Active Directory Integration HOWTO - ACB 20/01/13

## Todo
* Enable renaming wiki pages - JC
* Fix external links - Arran CB - Done 31/05/11 (see here https://github.com/github/gollum/pull/166)
* Fix per page committer info - Arran CB - Done 22/06/11 
* Add google as omniauth provider - Arran CB
* Implement proper fix for page creation - Arran CB - Done 20/06/11
* Fix TOC for markdown pages - Arran CB - Done 20/06/11
* Set up anonymous git access for wiki - Alan D
* Set up commit log to user mailing list - Alan D