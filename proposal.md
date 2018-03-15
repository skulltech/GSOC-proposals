## About Me

__Name.__ Sumit Ghosh  
__MB Username + IRC Nickname.__ SkullTech  
__Github.__ [SkullTech](https://github.com/SkullTech)  
__Website + Blog.__ [https://sumit-ghosh.com](https://sumit-ghosh.com)  
__LinkedIn.__ [Link](https://www.linkedin.com/in/sumit-ghosh-skulltech/)  
__E-mail.__ sumit@sumit-ghosh.com  
__Time-zone.__ UTC +5:30  

# Proposal

### Project Overview

As of now, BookBrainz only supports adding data to the database manually through the web interface. Building a database of considerable size only by this method would be highly inefficient. To solve this problem, we could develop a complete system for importing third-party data into the BookBrainz database easily. 

The following text taken from [here](https://wiki.musicbrainz.org/Development/Summer_of_Code/2018/BookBrainz) sheds light on the plan developed my _Leftmost_ and _Sputnik_ regarding this matter.

> At last year's summit, the two BookBrainz lead developers, Leftmost and LordSputnik worked on a plan for importing third party data into BookBrainz. This plan has several stages. First, data sources need to be identified, including mass import sources with freely available data, such as libraries, and manual import sources, such as online book stores and other user-contributed databases. The next stage is to update the database to introduce an "Import" object, which can be used to distinguish mass imported data from (usually better quality) user contributions. Then, actual import bots for mass import and userscripts for manual import will need to be written. Finally, it would desirable (but not necessary if time is short) to introduce an interface to the BookBrainz site to allow users to review automatically imported data, and approve it.


On the database side, a new table for storing the sources will be created. It will store an identifying name of the source, along with corresponding URL and description. A rough schema of the table is outlined below.

```sql
CREATE TABLE bookbrainz.source (
	id SERIAL PRIMARY KEY,
	name VARCHAR(64) NOT NULL UNIQUE CHECK (name <> ''),
	url VARCHAR(64) NOT NULL DEFAULT '',
	description TEXT NOT NULL DEFAULT '',
);
```

As we will be creating new entities from the imported data, we will need to specify the source in the stored data. For that, we will be modifying the entity table, adding two additional fields to it. One of the fields will indicate whether the entity is an imported one or user-contributed one. The other will point to the source in the case it is an imported one. Corresponding schema is outlined below. 

```sql
CREATE TABLE bookbrainz.entity (
	bbid UUID PRIMARY KEY DEFAULT public.uuid_generate_v4(),
	type bookbrainz.entity_type NOT NULL,
	imported BIT NOT NULL DEFAULT 0,
	source_id INT,
	CHECK ((source_id IS NOT NULL OR OR imported IS 0) AND (source_id IS NULL OR imported IS 1))
);
ALTER TABLE bookbrainz.entity ADD FOREIGN KEY (source_id) REFERENCES bookbrainz.source (id);
```

Here we added a check statement in the definition of entity, which takes care of the interdependent constraints between `source_id` and `imported`.