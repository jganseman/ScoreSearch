# ScoreSearch
Search for scores by instrumentation - HAMR2018 project
Find the latest version of this project at https://github.com/jganseman/ScoreSearch

## Summary
You and 2 friends team up to form a new band. You play the piano and your friends play bassoon and bagpipe. Who the hell wrote music for that combo? 

Well, surely someone unknown composer in some country wrote something once, but it's virtually impossible to find. Libraries generally record composer names, piece titles, even the physical size and the number of pages of the actual book very well. They rarely put the same effort in the instrumentation, and if they do, their software rarely allows to search it properly.

Hence this project that I've been willing to do for years: a search engine for sheet music by instrumentation. Currently only as a mock-up, worked out at the HAMR hackathon in Paris, September 2018.

## In this repo
A few scripts to get it done:
* `preprocess_rism.ipynb` : clean the RISM dataset and convert it to JSON files focusing on instrumentation
* `import_in_elk.ipynb` : import the resulting JSON into an ELK stack (i.e. ElasticSearch)
* TODO: script to launch the frontend / search form. (I'm counting on Kibana)

## Background

### The data
There are few large-scale repositories of bibliographic data readily available online. Most only offer limited access through an API, hardly useful to create a new index. Worldcat unfortunately does not share their data.

We start out with the RISM dataset, containing approximately 1 million bibliographic records and author data, of music written or published up until approximately 1850. Their data is kindly downloadable for free in MarcXML format from https://opac.rism.info/index.php?id=8&L=1 . If anybody knows about other datasets, do let it know, e.g. through a pull request!

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

Lacking a generally accepted standard sufficient in scope and depth, many libraries improvised their own systems for instrumentation catalography - see http://visiepc16.cde.ua.ac.be/varia/IAML2015_JG.pdf for an example. Add to that the unfortunate fact that catalographers rarely have the necessary (musical) skills to properly encode instrumentation. Obviously, this screws up all interchangeability. Instrumentation catalography being complicated, it's also expensive to migrate to another system.

## Pipeline

### Data cleansing
The most important data in RISM are:
* field 001: a unique identifier of a published musical work. The corresponding record can be accessed at https://opac.rism.info/id/rismid/<UUID>
* field 240m: a (standardized? To be confirmed...) summary of the instrumentation (e.g.: 1 piano plus strings)
* field 594a: a complete instrumentation (e.g.: 1 piano, 2 violins, 1 viola, 1 cello)

Extracting the used vocabulary in fields 240 and 594 from RISM gives list of instruments that includes things such as:
* "3. (3. auch Picc.)"
* "5-6). - Bühnenmusik Org."
* "Kast. grRatsche - 2 Hfe."
* "pf 4hands (clav 4hands)"
* "vl (fl 2)"

Needless to say, instrumentation is recorded in this database in a very ad hoc manner, often not following any standard, nor having used any form of input verification. RISM is a composite collection of data compiled from many libraries, of which few fully comply to international standards of catalography. 

To somewhat clean this up, we remove invalid characters (\[\]?\*+\& etc) and impose that:
* instruments must appear at least 5 times (configurable)
* instrument names must be maximum 20 characters (configurable)

### Format Conversion
All is converted into one .json file containing an array of JSON objects, each of the form:
```
{
  'id': '000051654', 
  'instr_summ': ['S', 'strings'], 
  'instr_full': ['S', 'vl (2)', 'vla', 'b']
}
```

### Database
This is basically an inverted index, which is apparently something that Lucene is good at. The ELK stack sounds (from what I've heard) like a solution that should be suitbale for these kind of problems. In this project, we locally use the Docker image: https://elk-docker.readthedocs.io/ . 
Make sure to increase the RAM available to Docker if necessary - 4GB is needed at least.

To install for the first time, open a terminal and run: 
`sudo docker run -p 5601:5601 -p 9200:9200 -p 5044:5044 -it --name elk sebp/elk`

If already installed, open a terminal and run:
`sudo docker start elk`

### Front-end
Since we installed an ELK stack, it's probably best to use Kibana - go to: http://localhost:5601 in the browser. 
I'll probably follow the tutorial at https://www.elastic.co/guide/en/kibana/6.4/tutorial-build-dashboard.html to create a dashboard that includes a search form.

## TODO
* Better cleanup of the data. We probably want to convert all cryptic acronyms into full-text instrument names, but this will be a long mapping to define
* Better exception handling in the code, e.g. when files are already imported in the DB, the index already exists, etc.
* Split up the code and files better in different functions with easily changeable parameters
* Frontend definition in Kibana
* Undoubtedly forgot a few special cases
* Long term: find a way to separate the number of instruments from the instrument names themselves

## To think about
* What does the number of instruments actually mean: the number of performers, or the actual number of instruments needed? How is piano 4handed different from 2 pianos? What with an orchestra where the piccolo and flute are played by the same person?
* Exclusive search should probably be a tickbox in the search form: music for piano, violin and cello automatically includes piano concertos since orchestras have violins and cellos, and this is likely not always what we want.
* Adding more databases. E.g. is the IMSLP data accessible? And where to find modern composers?
