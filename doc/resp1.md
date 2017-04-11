I've divided these responses into three classes: Points which I definitely will
respond to ('shall do'); those which I would like to yet remain uncertain how
best to ('would like to do'); and those with which I will remain in disagreement
until potentially convinced otherwise ('shall not do').

## shall do

First start with the easy bits which i oughta coulda shoulda done better and
will certainly do so in response to @edzer's comments:

1. `NULL` assignment in function def rather than verbose re-def of `missing ->
   NULL`: Shall do
2. Class definition: Yeah, you're right that a simple `structure (list(), class
   = 'osmdata')` would and should replace the entire redundant class def. Shall
   do.
3. Claim that it's confusing that `osmdata_sf (q, doc)` writes *from* `doc` into
   `sf`, yet `osmdata_xml (q, doc)` writes *to* `doc` - um, yeah, i hadn't
   thought of it that way, but that could certainly be somewhat confusing. I am,
   however, unsure of how to resolve this. I like the simplicity of a mere three
   main functions all of form `osmdata_abc()`, with similar arguments in all
   cases. All functions can or do return data of the form specified after that
   subscript, so that's consistent. The only inconsistency is that
   `osmdata_xml()` also enables `doc` to be *written*. Making this any neater
   would likely require adding an additional step, so `osmdata_xml()` would
   purely serve to import into `R`, with an additional stage required to
   potentially write back to `xml`. I'm not sure what would be less confusing:
   the admittedly present yet likely quite mild confusion highlighted here, or
   unnecessarily breaking a currently very simple one-liner into two distinct
   functions? I simply don't know at present, but would welcome any suggestions.
4. Message produced on `quiet = TRUE` - yep, ought not be there and shall be
   removed.
5. Return of `osmdata_sp` - yeah, you're right, it would be better to use strict
   `sp` classes, so will modify to only declare `sp` classes of
   `SpatialLines/PolygonsDataFrame`. `osmdata` objects will nevertheless remain
   divided into the two respective `osmdata` types of `lines / multilines` and
   `polygons / multipolygons`. This will obviously demand improved clarification
   both in the `osmdata_sp()` description itself and the vignette. The lack of
   distinction in `sp` between `lines / multilines` and `polys / multipolys` is
   a non-issue, because the OSM itself makes a fundamental distinction between
   these two to which `osmdata` adheres (as elaborated on below).
6. `configure` message re: `unexpected operator` - yeah, that's no longer
   required for `codecov` anyway, so i can and will fix that.
7. Link to claim of being faster than `sf/GDAL` is dead: Whoops, yeah, shall fix
   that.
8. Link to `sfa` in vignette#2: Thanks, shall fix.

## would like to do

Now on to those points regarding which I am uncertain how best to respond.

1. 
   > It seems that `q1` (in the call `x <- osmdata_sf(q1, "data.xml")`) is added
      to the return object which takes its data from "data.xml" 

   Yes, that's correct.
   > but there is no guarantee that the two belong together.

   There are abundant checks within `osmdata_sf()` to ensure the forms and
   classes of those objects are appropriate to the call, with extensive tests
   throwing a range of incompatible objects in there, and generating
   appropriately informative messages. 
   > When the query call is added to the returned object
   (e.g. by osmdata_xml) this guarantee can be given, and ambiguity is excluded.
   
   The query call, `q1` is added to the returned object, so i am unsure what to
   make of this.
2. 
   > It would help to memorize what [the functions] do if [the function names]
   contain verbs
   
   yeah, i entirely agree as a general point, but as mentioned above, I also
   favour the simplicity of `osmdata_abc()` for the three primary functions.
   Extra verbs would likely best be incorporated by simply renaming these to
   `osmdata_get_abs()`, but I don't really see that as adding anything
   constructive. I'll nevertheless have a ponder in regard to other function
   names.
3. The very important question raised just before the list of 'Minor things':
   > Do `R` users need to learn the OSM Overpass API query call language?
   
   I wish I could give a clear answer to this, but alas can not. The `osmdata`
   package is an attempt to facilitate direct access to OSM data without the
   need to understand this sometimes non-trivial query language (QL). It comes
   down to a compromise between breadth and depth. We've aimed for a good
   breadth, and so hope the vast majority of typical use cases can be executed
   just by examining the `osmdata` documentation and forgetting entirely about
   `overpass`. The depth has hopefully nevertheless been retained so truly
   complex queries nevertheless remain possible. Do `R` users need to learn the
   QL? I would certainly hope not. Would learning the QL enable more powerful
   use of `osmdata`? I would certainly hope so. Where can we best strike the
   balance there between? ... I don't know, but hope we've given it a good shot.

## shall not do

1. 
   > it would be good to also mention that osmdata reads openstreetmap vector
    data, as opposed to e.g. package OpenStreetMap which imports openstreetmap
    raster data (images).

   I don't agree here because `osmdata` simply reads data from OpenStreetMap in
   a form that reflects as accurately as possible the underlying form of OSM,
   which is a vector form. Raster images have nothing to do with OSM *data* -
   they are only used by OSM to generate tiles for their web interface, which in
   turn merely serves to visualise the underlying **data**.  That's why the
   package is called `osmdata` and not `osm` (or whatever). It's about the
   **data**. The package `OpenStreetMap` is in no way directly connected with
   OSM, and does not directly deliver OSM **data**; it merely renders them in
   visual form.
2. ''Performance, comparison'' and speed: With due apology for the dead link
mentioned above, the presented figures nevertheless reveal that `osmdata` is
indeed >20% faster than `sf/GDAL`. Is `GDAL` very fast? I'd think so. More
importantly, are either `sf` or `osmdata` very fast in comparison to the
long-standing singular alternative for getting OSM data into R? I can't even
given numbers, but the answer is yes by factors of thousands. Both `sf/GDAL` and
`osmdata` are very fast, yet `osmdata` remains nevertheless the fastest. Surely
that's a justification for claiming ''very fast''? Compared to all other
currently possible alternatives, it is so.
3. Changing the `config` file of the `GDAL` OSM driver:
Other than explicit customisation of which `key` fields ought to be returned,
the only options that the `GDAL config` file enables are potentially
returning all points and all ways. There still remains no way within
`sf/GDAL` to connect any of these ways (whether `line` or `polygon` objects)
to the `multi..` objects of which they may or may not be members. `osmdata`
retains all such membership, and through the recursive searching functions
enables immediate reconstruction of all members of any given object, or of
all objects which share a set of members. That is simply not possible with
`sf/GDAL`, no matter how much the `GDAL config` file is customised. In short,
there is no `GDAL config` setting that can reconstruct how `osmdata` extracts
data, and i stand by the claims of our second vignette that `osmdata` remains
generally ''truer'' to the underlying structure of OSM data (with due
acknowledgement in that vignette that the entire `SF` scheme was never intended
for such purposes, so that ought not be interpreted as a criticism!).
3. 
   > I wonder, for an end-users, what the value is of getting a point set that
   consists of not only point features (such as traffic lights, or way signs)
   but also all vertices in lines and polygons. 

   The package simply aims to bring OSM data into `R` in a form that resembles as
closely as possible the entire philosophy and structure of OSM data itself.
Points would only be removed because of some arbitrary decision on our part that
they are somehow not as important as other data components - it is an assuredly
far more neutral design decision to simply leave the data in as intact a form as
possible, and thereby enable the widest possible range of possible usage.
