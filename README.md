# sling-jbake
Experimenting with JBake for the Apache Sling website.

See also https://issues.apache.org/jira/browse/SLING-6955

## How to publish the website
Build the site as described below, and sync the ./output folder to the asf-site branch 
of this Git repository. The ASF's gitpubsub mechanism then synchronizes that content
on the http://sling.apache.org website.

gitpubsub is active at https://sling.apache.org/ng/ for testing but as pages now contain some absolute links, navigation might lead back to the existing site (https://issues.apache.org/jira/browse/INFRA-14390)

## How to build the site locally
* Clone this repository
* Run `./bake.sh`
* Open http://localhost:8820/ and enjoy.

## Publishing on the live website
The live content is synced from the `asf-site` branch of this repository. To publish, generate the content and sync the `output` folder at the root of that branch.

We'll have a script for that, but for now the following works:

    ./bake.sh
    
    # verify locally staged content at http://localhost:8820/
    
    # if ok, commit your changes to the master branch
    
	# if ok, set these values in jbake.properties
	# and generate again
	# (temporary, while testing on sling.apache.org/ng)
	site.host=http://sling.apache.org	
	site.contextPath=/ng/

    ./bake.sh
    
    git checkout -- jbake.properties
    mv ./output /tmp
    git checkout asf-site
    rsync -r /tmp/output/* .
    git commit -a -m "Updating live site"
    
    # content should appear at https://sling.apache.org/ng/ (for now) soone

## TODO

### Remaining fixes as of July 26th, 2017
* HTML page title is "Apache Sling on JBake", should be the page title.

### apache.org requirements
* Activate all the required links checked by https://whimsy.apache.org/site/

### Nice to have
* JBake 2.5.x does not support the `[TOC]` macro but apparently that will be available once JBake moves to https://github.com/vsch/flexmark-java , probably in its next version. We might wait for that and just mark the TOCs as unsupported for now.
* The `#!xml` and `#!java` code higlighting macros are not supported. I have left them in place for future automatic translation, once we reactivate code highlighting (using https://highlightjs.org/ maybe?)

### Final validation, activation etc.
* Review all pages
* Resync the content with the current Sling website if needed, initially synced at r1798604
* Breadcrumbs. In http://sling.apache.org/documentation/bundles/rendering-content-default-get-servlets.html for example, provide `Home :: Documentation :: Bundles` links.

### Done
* Fix internal links like `refs.project-information.path` 
* Page header and footer, logo etc
* Remove unused assets files and templates (copied from JBake Groovy sample)
* Tables work now, needed the pegdown TABLES extension
* Move images and other files to /assets and convert their links
* Fix references like `refs.authentication-tasks.headers.excerpt` as well as `.title` references -> replace with copies of those titles and excerpts, or use tags to mark and select
* Fix remaining `refs.` links -> those are probably broken anyway
* Test the downloads.cgi page, requires an apache.org setup (INFRA-14390) -> done at https://sling.apache.org/ng/downloads.cgi
* Fix broken tables: the current pegdown parser has troubles with table cells containing special characters, tables containing a single dash for example need to be converted to `(-)` as a workaround.
* Enumerate child pages in documentation/tutorials-how-tos.html and smilar pages -> replaced with a manually generated list.
* Sitemap page
* Left menu layout is now correct

## JBake and other techn notes
* Currently using 2.5.1, see under `/bin`, docs at http://jbake.org/docs/2.5.1
* Uses https://github.com/sirthias/pegdown for Markdown, syntax info at https://github.com/sirthias/pegdown/blob/master/src/test/resources/MarkdownTest103/Markdown%20Documentation%20-%20Syntax.md , extensions at http://www.decodified.com/pegdown/api/org/pegdown/Extensions.html
* Groovy MarkupTemplateEngine examples at https://github.com/jbake-org/jbake-example-project-groovy-mt , docs for that engine at http://groovy-lang.org/templating.html#_simpletemplateengine
* Other Apache projects using JBake include at least Tamaya and OpenNLP and the Incubator is apparently also switching to it.

## Useful scripts and commands
To find broken links use 

    wget --spider -r -nd -nv -l 5 http://localhost:8820/ 2>&1 | grep -B1 'broken link'

To find leftover `refs.` in pages use

    wget -r -nv -l 5 http://localhost:8820/
    find localhost\:8820/ -type f | xargs grep -l 'refs\.'
