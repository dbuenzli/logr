logr — Static ring web logger
-----------------------------------------------------------------------------

logr is a simple web logger that publishes time sorted, static, log
entries as a ring: from one entry you only get to the next or the
previous one, through the *index* when no such element exists. The
index has random access to any elements of the ring and is accessible
from every entry.

logr is a BSD3 licensed bloody shell script. It should be rewritten in
a decent programming language. Why did you do that ?

## Install prerequisites

    opam install uuidm omd unison
    
## Make a log 

    git clone http://erratique.ch/repos/logr.git mylog
    cd mylog
    
The contents of this directory is:

    README.md        # the bytes you are reading
    logr             # the driver script
    log/config       # a few configuration variables
    log/template     # templating script
    log/feed         # feed templating script
    log/build        # build subscript
    log/content.md   # content placeholder for a new entry
    assets/*         # files copied at the root of the log
    _build           # holds builds can be rm -rf'd safely
    $YYYY-$MM-$DD    # directories for log entries

As you make new entries directories of the form `YYYY-MM-DD` will be 
created.

The first thing you need to do is to generate an UUID with by invoking
`uuidtrip` and paste the result in the `LOG_UUID` variable of
`log/config`. You may also want to adjust the other variables there
and tweak the `log/template` and `log/feed` template files.

## logr tool synopses

    ./logr new         # make a new entry (see below)
    ./logr update DIR  # updates an entry (see below)
    ./logr clean       # removes _build directroy 
    ./logr build       # builds the log
    ./logr publish     # builds the log and syncs with remote

## Build structure and local preview

The build directory has:

    _build/index.html                # generated web log index 
    _build/feed.atom                 # generated atom feed
    _build/sitemap.xml               # generated search engine sitemap
    _build/*                         # files from assets/* 
    _build/$YYYY-$MM-$DD             # entries
    _build/$YYYY-$MM-$DD/index.html  # generated entry 
    _build/$YYYY-$MM-$DD/*           # some files from $YYYY-$MM-$DD/* 
    
If you have a webserver running on your machine. You can preview a build
with: 

    ln -s _build $DOCUMENTROOT/mylog 
            
# Make a new entry

Each entry is in a directory named after a time stamp. Invoking

    ./logr new 

Creates a directory `YYYY-MM-DD[.XX]` where `.XX` is an optional
increasing number if there's already a directory with that name.

The contents of this directory is:

    log/stamp  # RFC 3339 timestamp of log new 
    log/update # RFC 3339 timestamp of last log update (possibly)
    nolog/*    # unpublished files (possibly)
    content.md # markdown file holding the log entry 
    *.js       # javascript file automatically added to <meta> (possibly)
    *.css      # css files automatically added to <meta> (possibly)
    *          # any other file for publication (possibly)
    
## Update an entry 

If you need to update an entry `YYYY-MM-DD` after it was published,
make your changes and then invoke

    ./logr update YYYY-MM-DD
    
This will simply add a time stamp in `YYYY-MM-DD/log/update` that
will be used in the atom feed appropriately. 

## Implementation hacks and details to know 

* The title of an entry is generated by taking the very first line 
  of `content.md`, removing a potential initial markdown `#` and
  passing it through `omd -notag`.
* The $LOG_TITLE in `log/config` is not properly escaped.
* The contents of an entry is generated by removing the very first
  line of `content.md` and passing the rest to `omd`.
* Feed entry uuids are sha1 namespace UUIDs in the namespace of `LOG_UUID`
  defined in `log/config`. The name is the initial time stamp of the 
  entry stored in its `log/stamp` file. This is why once published the 
  original `log/stamp` of an entry should not be changed anymore otherwise 
  the feed entry id will change.
* We are using `date` to generate the time stamps and (on osx) we
  cannot get to subsecond precision so dont invoke `log new` twice
  during the same second otherwise you will have two entries with the
  same uuid. We could add a `sleep 1` to guarantee that. 
