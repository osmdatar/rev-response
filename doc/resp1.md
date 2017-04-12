Thanks @edzer for the useful comments and your detailed consideration of
`osmdata`! :smile_cat: These responses are divided into into two classes: Points which we
can easily respond to and in some cases already have and those for which we feel more discussion (with the reviewer) and thinking (on our side) is needed.

Note that collective pronouns refer here to @mpadge, @RobinLovelace, and
@maelle, unless otherwise explicitly indicated.

## easy to solve issues :sunglasses:

First start with the easy bits :cake::

1. `NULL` assignment in function def rather than verbose re-def of `missing ->
   NULL`: Already done; thanks!
3. Message produced on `quiet = TRUE` - yep, ought not to have been there and
   has already been removed.
4. Return of `osmdata_sp` - yeah, you're right, that was an inappropriate
   mis-interpretation of `sp`, and has already been rectified.  `osmdata`
   objects nevertheless remain divided into the two respective `osmdata` types
   of `lines / multilines` and `polygons / multipolygons`, with `sp` forms now
   simply declaring these as `SpatialLines` and `SpatialPolygons` regardless of
   `multi_` or not. The lack of distinction in `sp` between `lines / multilines`
   and `polys / multipolys` is a non-issue, because the OSM itself makes a
   fundamental distinction between these two to which `osmdata` adheres.
5. `configure` message re: `unexpected operator` - yeah, that's no longer
   required for `codecov` anyway, so will be fixed.
6. Link to claim of being faster than `sf/GDAL` is dead: Already fixed - thanks!
7. Link to `sfa` in vignette#2: Fixed - thanks once more!

## open issues pending further discussion / debate :construction_worker:

Now on to those points regarding which we are uncertain how best to respond. :chestnut:

### Code 
1. Class definition: we entirely agree with you here, but are unsure how best to
   resolve the issue. While a simple `structure (list(), class = 'osmdata')` is
   a great suggestion, the class def is strictly required (as far as we all
   understand it) in order to provide the `print.osmdata` and `c.osmdata`
   functionality. These are both utterly necessary: The first to avoid all data
   being dumped to screen; and the second because it's enormously useful for
   combining query results within `R`. If you - or anyone else - can suggest
   ways to effectively `print` and `c` without defining a class, we'd be more
   than happy to remove the class def entirely.
   
2. 
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
   
   The query call, `q1` is added to the returned object, so i (MP) am unsure
   what to make of this.
3. 
   > It would help to memorize what [the functions] do if [the function names]
   contain verbs
   
   yeah, we entirely agree as a general point, but as mentioned above, we also
   favour the simplicity of `osmdata_abc()` for the three primary functions.
   Extra verbs would likely best be incorporated by simply renaming these to
   `osmdata_get_abc()`, but we don't really see that as adding anything
   constructive. We'll nevertheless have a ponder in regard to other function
   names.
   
4. Comment that it's confusing that `osmdata_sf (q, doc)` writes *from* `doc` into
   `sf`, yet `osmdata_xml (q, doc)` writes *to* `doc` - um, yeah, i (MP) hadn't
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


### Package functionality


1.
   > osmar ... can do shortest paths through a network, and
   exposes the street network to R users (combining sp and igraph), osmdata does
   not expose the network.

   That's true, and mainly because our current vision of `osmdata` is as a
   coherent package that serves the singular purpose of getting OSM data into
   `R` in a way that most faithfully represents the structure and intent of OSM
   itself. 'Networks' are not a part of OSM - they are a post-imposition by
   others wishing to use OSM for routing. We're simply not sure at present
   whether such arguably non-OSM functionality belongs in this package. A
   function to 'expose the network' has already been constructed as described in
   [this `osmdata` issue](https://github.com/osmdatar/osmdata/issues/39).
   Potential incorporation depends on the ever-evolving discussions of the
   extent to which `R` packages should best encapsulate singularly focussed
   functionality, or broader ranges thereof. The upcoming fragmentation of
   `devtools` into numerous individual packages certainly reifies Hadley's
   view on that, by which we're inclined to abide.
   
2. The very important question raised just before the list of 'Minor things':
   > Do `R` users need to learn the OSM Overpass API query call language?
   
   We wish we could give a clear answer to this, but alas can not. The `osmdata`
   package is an attempt to facilitate direct access to OSM data without the
   need to understand this sometimes non-trivial query language (QL). It comes
   down to a compromise between breadth and depth. We've aimed for a good
   breadth, and so hope the vast majority of typical use cases can be executed
   just by examining the `osmdata` documentation and forgetting entirely about
   `overpass`. The depth has hopefully nevertheless been retained so truly
   complex queries nevertheless remain possible. Do `R` users need to learn the
   QL? We would certainly hope not. Would learning the QL enable more powerful
   use of `osmdata`? We would certainly hope so. Where can we best strike the
   balance there between? ... We don't know, but hope we've given it a good shot.
  
3. 
   > it would be good to also mention that osmdata reads openstreetmap vector
    data, as opposed to e.g. package OpenStreetMap which imports openstreetmap
    raster data (images).

   We don't agree here because `osmdata` simply reads data from OpenStreetMap in
   a form that reflects as accurately as possible the underlying form of OSM,
   which is a vector form. Raster images have nothing to do with OSM *data* -
   they are only used by OSM to generate tiles for their web interface, which in
   turn merely serves to visualise the underlying **data**.  That's why the
   package is called `osmdata` and not `osm` (or whatever). It's about the
   **data**. The package `OpenStreetMap` is in no way directly connected with
   OSM, and does not directly deliver OSM **data**; it merely renders them in
   visual form.

4. 
   > I wonder, for an end-users, what the value is of getting a point set that
   consists of not only point features (such as traffic lights, or way signs)
   but also all vertices in lines and polygons. 

   The package simply aims to bring OSM data into `R` in a form that resembles as
closely as possible the entire philosophy and structure of OSM data itself.
Points would only be removed because of some arbitrary decision on our part that
they are somehow not as important as other data components - it is an assuredly
far more neutral design decision to simply leave the data in as intact a form as
possible, and thereby enable the widest possible range of possible usage. Points
that are part of other objects can be readily 'filtered' out through judicious
use of `c.osmdata` functionality. More importantly, OSM demands unique ID labels
on all components so that they can always be connected together: all points
within a line, all lines which contain any given point, all whatever which are
within or contain bits of whatever else. We see no need to remove or reduce the
ability of `osmdata` to transfer as directly as possible such abilities to
assemble and dissemble any and all data components within an `R` environment.
`osmdata` brings OSM data into `R` it its entirety; the `GDAL` OSM driver does
not extract complete data because even with `config` set to full volume, `GDAL`
strips all object IDs, and so prevents any ability to assemble and dissemble
geometrically distinct components.

### Comparison with `GDAL`

1. ''Performance, comparison'' and speed: With due apology for the dead link
mentioned above, the presented figures nevertheless reveal that `osmdata` is
indeed >20% faster than `sf/GDAL`. Is `GDAL` very fast? Surely so. More
importantly, are either `sf` or `osmdata` very fast in comparison to the
long-standing singular alternative for getting OSM data into R? We can't even
given numbers, but the answer is yes by factors of thousands. Both `sf/GDAL` and
`osmdata` are very fast, yet `osmdata` remains nevertheless the fastest. Surely
that's a justification for claiming ''very fast''? Compared to all other
currently possible alternatives, it is so. And finally, 20% faster than the
current fastest surely represents an improvement?
8. Changing the `config` file of the `GDAL` OSM driver:
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
data, and we stand by the claims of our second vignette that `osmdata` remains
generally ''truer'' to the underlying structure of OSM data (with due
acknowledgement in that vignette that the entire `SF` scheme was never intended
for such purposes, so that claim ought not be interpreted as a criticism!).

## Thankfulness

Note that we consciously didn't acknowledge @edzer's contributions prior to review but will definitely do it after review with a double role:

* `cph` of `sf` code :sparkles:

* `rev` because of the improvements of the package thanks to his review :ok_hand:
