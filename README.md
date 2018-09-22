# ScoreSearch
Search for scores by instrumentation - HAMR2018 project

### Summary
You and 2 friends team up to form a new band. You play the piano and your friends play bassoon and bagpipe. Who the hell wrote music for that combo? 

Well, surely someone did, but currently it's virtually impossible to find. Libraries generally record composer names, piece titles, even the physical size and the number of pages of the actual book very well. They rarely put the same effort in the instrumentation, and if they do, their software rarely allows to search it properly.

Therefore this project that I've been willing to do for years: a search engine for sheet music by instrumentation. Currently only as a mock-up, worked out at the HAMR hackathon in Paris, September 2018.

### In this repo
A few scripts to get it done:
* cleaning the data
* converting it to the format we'll use in our database
* moving the data into the database
* launching the frontend / search form

### The data
There are few large-scale repositories of bibliographic data readily available online. Most only offer limited access through an API, hardly useful to create a new index. Worldcat unfortunately does not share their data.

We start out with the RISM dataset, containing approximately 1 million bibliographic records and author data, of works published up until approximately 1850. Their data is kindly downloadable for free in MarcXML format from https://opac.rism.info/index.php?id=8&L=1 . If anybody knows about other datasets, do let it know, e.g. through a pull request!

The MARC21 and MarcXML formats are standard formats for bibliographic data, of which the (very complicated) definition can be found at the Library of Congress: http://www.loc.gov/standards/marcxml/

Just build a reverse index on that RISM file, easy peasy right? Well...

### The road is long and winding
There exists no internationally agreed upon list or taxonomy of musical instruments. Or better, there exist multiple: 
* General MIDI is heavily skewed towards electronic music/effects: https://soundprogramming.net/file-formats/general-midi-instrument-list/
* MARC21 contains a Musical Instruments and Voices Code List: https://www.loc.gov/standards/valuelist/marcmusperf.html . It is more balanced, but with 99 2-letter codes, only offers basic functionality.
* IAML defines a Medium of Performance list as part of the UNIMARC data format: https://www.iaml.info/de/unimarc-field-146-medium-performance . It contains 281 3-letter codes that represent 655 instruments. UNIMARC as a data format is however only common in French-speaking countries.
* In 2014 the Library of Congress proposed a new list with over 800 instrument names, LCMPT: https://www.loc.gov/aba/publications/FreeLCMPT/freelcmpt.html . Whether it will be adopted remains to be seen, as this comes way too late.
* Needing it to index CD recordings, MusicBrainz defined their own list of instruments: https://musicbrainz.org/instruments . This is probably one of the first fairly complete ones. 
* Efforts to create musical instrument taxonomies/ontologies for the Semantic Web include work by Kolozali et al.: http://www.mirlab.org/conference_papers/International_Conference/ISMIR%202011/papers/PS3-19.pdf

Note that most of these are also heavily biased towards the Western musical tradition.

Lacking a generally accepted standard sufficient in scope and depth, many libraries improvised their own systems for instrumentation catalography - see http://visiepc16.cde.ua.ac.be/varia/IAML2015_JG.pdf for an example. Add to that the unfortunate fact that library catalographers rarely have the necessary (musical) skills to properly encode instrumentation. Obviously, this screws up all interchangeability. Instrumentation catalography being complicated, it's also expensive to migrate to another system.

### Data cleansing
The most important data in RISM are:
* field 001: a unique identifier of a published musical work. The corresponding record can be accessed at https://opac.rism.info/id/rismid/<UUID>
* field 240m: a (standardized? To be confirmed...) summary of the instrumentation (e.g.: 1 piano plus strings)
* field 594a: a complete instrumentation (e.g.: 1 piano, 2 violins, 1 viola, 1 cello)

Extracting the used vocabulary in fields 240 and 594 from RISM gives list of instruments that includes:
* bla
* bla
* bla

RISM is a collection of data gathered from many libraries that rarely comply to international standards of catalography, making the instrumentation data an utter mess. 

To somewhat clean this up, we remove invalid characters (\[\]?\*\& etc) and impose that:
* instruments must appear at least 5 times 
* instrument names must be maximum 20 characters

### Format Conversion
Directly to JSON for moving into Elastic, or rather to structured ASCII for use with LogStash?

### Database
Inverted index, should be something Lucene can do, so: ElasticSearch / ELK stack. Probably easiest on Docker?

### Front-end
Kibana to begin with? Should be possible to make a form in it, at least their demo has one.
