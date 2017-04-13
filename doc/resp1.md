Thanks @edzer for the useful comments and your detailed consideration of
`osmdata`! :smile_cat: These responses are divided into into two classes:

- Points which we can easily respond to and in some cases already have.
- Those for which we feel more discussion (with the reviewer and potentially
  others) and thinking (on our side) is needed.

Note that collective pronouns refer here to @mpadge, @RobinLovelace, and
@maelle, unless otherwise explicitly indicated.

## easy to solve issues :sunglasses:

First start with the easy bits :cake:, mostly already done with 
[this commit](https://github.com/osmdatar/osmdata/commit/b0d304c7722553fdb983d242c6032b1875a74d4f):

1. `NULL` assignment in function def rather than verbose re-def of `missing ->
   NULL`: Already done; thanks!
3. Message produced on `quiet = TRUE` - agreed, ought not to have been there and
   has already been removed.
4. Return of `osmdata_sp` - agreed: an inappropriate
   mis-interpretation of `sp`, and has already been rectified.  `osmdata`
   objects nevertheless remain divided into the two respective `osmdata` types
   of `lines / multilines` and `polygons / multipolygons`, with `sp` forms now
   simply declaring these as `SpatialLines` and `SpatialPolygons` regardless of
   `multi_` or not. The lack of distinction in `sp` between `lines / multilines`
   and `polys / multipolys` is a non-issue, because the OSM itself makes a
   fundamental distinction between these two to which `osmdata` adheres.
5. `configure` message re: `unexpected operator` - agreed, that's no longer
   required for `codecov` anyway, so will be fixed.
6. Link to claim of being faster than `sf/GDAL` is dead: Already fixed - thanks!
(RL: I hope to add a benchmark comparing with `osmar` which in my experience is
painfully slow. The example benchmark is useful, would be interesting to build
on it by a) downloading some data also and b) comparing against larger
datasets.)

7. Link to `sfa` in vignette#2: Fixed - thanks once more!

## open issues pending further discussion / debate :construction_worker:

Now on to those points regarding which we are uncertain how best to respond. :chestnut:

### Code 

1. Class definition: we entirely agree with you here, but are unsure how best to
   resolve the issue. While a simple `structure (list(), class = 'osmdata')` is
   a great suggestion, the class def is strictly required (as far as we
   understand it) in order to provide the `print.osmdata` and `c.osmdata`
   functionality. These are both utterly necessary: The first to avoid all data
   being dumped to screen; and the second because it's enormously useful for
   combining query results within `R`. If you - or anyone else - can suggest
   ways to effectively `print` and `c` without defining a class, we'd be more
   than happy to remove the class def entirely. A solution is obviously strongly
   desired, because we agree entirely that it can be nothing other than
   confusing that the class constructor `osmdata()` is present yet neither can
   nor should be used.
   
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
   
   The query call, `q1` is added to the returned object, so I (MP) am unsure
   what to make of this. Note that `osmdata_xml()` really just returns the
   `.xml/.osm` data directly from `overpass` with no post-processing, and so is
   a fundamentally different function to `osmdata_sf()` and `osmdata_sp()`. The
   latter two really don't use or rely on the functionality of `osmdata_xml()`.
   
3. Comment that it's confusing that `osmdata_sf (q, doc)` writes *from* `doc` into
   `sf`, yet `osmdata_xml (q, doc)` writes *to* `doc`. I (MP) hadn't
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
4. 
   > It would help to memorize what [the functions] do if [the function names]
   contain verbs
   
   yeah, we entirely agree as a general point, but as mentioned above, we also
   favour the simplicity of `osmdata_abc()` for the three primary functions.
   Extra verbs would likely best be incorporated by simply renaming these to
   `osmdata_get_abc()`, but we don't really see that as adding anything
   constructive. We'll nevertheless have a ponder in regard to other function
   names.

### Package functionality

1.
   > osmar ... can do shortest paths through a network, and
   exposes the street network to R users (combining sp and igraph), osmdata does
   not expose the network.

   That's true, and mainly because our current vision of `osmdata` is as a
   coherent package that serves the singular purpose of getting OSM data into
   `R` in a way that most faithfully represents the structure and intent of OSM
   itself.
   
   We encourage others to add functionality building on `osmdata` for routing,
   e.g. building on the idea of `stplanr::SpatialLinesNetwork()` or a
   [`spnetwork`](https://github.com/edzer/spnetwork) package (RL).
   
   'Networks' are not a part of OSM - they are a post-imposition by
   others wishing to use OSM for routing (e.g. OSRM). We're unsure
   whether such arguably non-OSM functionality belongs in this package. A
   function to 'expose the network' has already been constructed as described in
   [this `osmdata` issue](https://github.com/osmdatar/osmdata/issues/39).
   Potential incorporation depends on the ever-evolving discussions of the
   extent to which `R` packages should best encapsulate singularly focussed
   functionality, or broader ranges thereof. The upcoming fragmentation of
   `devtools` into numerous individual packages certainly reifies Hadley's
   view on that, by which we're inclined to abide.  The concept that packages
   should 'do one thing unambiguously and do it well' is a sensible design
   philosophy we hope `osmdata` follows.
   
2. The very important question raised just before the list of 'Minor things':
   > Do `R` users need to learn the OSM Overpass API query call language?
   
   We cannot provide a clear answer to this question.  A analogous question
   would be 'can `sf::st_read()` be used/understood without learning GDAL?'. Yes
   it can be used and it mercifully makes life easier, but understanding the
   back-end code-base helps greatly.  We have tried to communicate this in the
   vignette but are open to ideas for improving the communication of this
   message.
   
   The `osmdata` package is an attempt to facilitate direct access to OSM data
   without the need to understand this sometimes non-trivial query language
   (QL). It comes down to a compromise between breadth and depth. We've aimed
   for a good breadth, and so hope the vast majority of typical use cases can be
   executed just by examining the `osmdata` documentation and forgetting
   entirely about `overpass`. The depth has hopefully nevertheless been retained
   so truly complex queries nevertheless remain possible. Do `R` users need to
   learn the QL? We would certainly hope not. Would learning the QL enable more
   powerful use of `osmdata`? We would certainly hope so. Where can we best
   strike the balance there between? ... We don't know, but we've given it a
   good shot.
  
3. 
   > it would be good to also mention that osmdata reads openstreetmap vector
    data, as opposed to e.g. package OpenStreetMap which imports openstreetmap
    raster data (images).

   We don't agree here because `osmdata` simply reads data from OpenStreetMap in
   a form that reflects as accurately as possible the underlying form of OSM,
   which is a vector form.  Raster images have nothing to do with OSM *data* -
   they are only used by OSM to generate tiles for their web interface, which in
   turn merely serves to visualise the underlying data.  That's why the package
   is called `osmdata` and not `osm` (or whatever). It's about the data. The
   `R` package `OpenStreetMap` is in no way directly connected with OSM, and
   does not directly deliver OSM data; it merely renders them in visual form.
   
   We nevertheless acknowledge in response to this concern the importance of 
   mentioning how `osmdata` differs from the very different `OpenStreetMap`
   package, in order to avoid cofusion around names, and have flagged this task
   in an [`osmdata` issue](https://github.com/osmdatar/osmdata/issues/59).

4. 
   > I wonder, for an end-users, what the value is of getting a point set that
   consists of not only point features (such as traffic lights, or way signs)
   but also all vertices in lines and polygons. 

   The package simply aims to bring OSM data into `R` in a form that resembles as
closely as possible the structure and design of OSM data itself.
Points would only be removed because of some arbitrary decision on our part that
they are somehow not as important as other data components - it is an assuredly
far more neutral design decision to simply leave the data in as intact a form as
possible, and thereby enable the widest possible range of potential usage.
OSM demands unique ID labels on all components so that they can always be
connected together: all points within a line, all lines which contain any given
point, all whatever which are within or contain bits of whatever else.  We see
no need to remove or reduce the ability of `osmdata` to transfer as directly as
possible such abilities to assemble and dissemble any and all data components
within an `R` environment.  `osmdata` brings OSM data into `R` it its entirety;
the `GDAL` OSM driver does not extract complete data because even with `config`
set to full volume, `GDAL` strips all object IDs, and so prevents any ability to
assemble and dissemble geometrically distinct components.

   Nevertheless, in response to this legitimate concern, we have decided on the
   following package modifications:
      - The primary `osmdata_...()` functions shall remain as they are and
        import **all** OSM data, to ensure that they remain as true as possible
        to the underlying data model of OSM itself; but
      - We will implement a new function (see 
         [this `osmdata` issue](https://github.com/osmdatar/osmdata/issues/60)
         that enables reduction of a given `osmdata` object to include only
         unique elements of each geometric class.  The result of this will then
         more closely resemble the output of GDAL, while retaining the primary
         difference described above that `osmdata` will still retain all
         individual OSM IDs for all components.

   We nevertheless note in direct response to the above concern that a use case
   would be routing to minimse numbers of traffic lights, which could only be
   done through storing all points as `point` objects in addition to their
   presence as line and polygon vertices.

### Comparison with `GDAL`

1. ''Performance, comparison'' and speed: this is a useful benchmark, which
I (RL) think a modified version of would be useful in the vignette to illustrate
speed and usage differences with the 'raw' way of doing things.

   With due apology for the now-rectified dead link mentioned above, the
   presented figures nevertheless reveal that `osmdata` is indeed >20% faster
   than `sf/GDAL`. Is `GDAL` very fast? Surely so.  More importantly, are either
   `sf` or `osmdata` very fast in comparison to the long-standing singular
   alternative for getting OSM data into R? We can't even given numbers, but the
   answer is yes by factors of thousands. Both `sf/GDAL` and `osmdata` are very
   fast, yet `osmdata` remains nevertheless the fastest. Surely that's a
   justification for claiming ''very fast''? Compared to all other currently
   possible alternatives, it is so. And finally, 20% faster than the current
   fastest surely represents an improvement?

2. Changing the `config` file of the `GDAL` OSM driver:
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

Note that we consciously didn't acknowledge @edzer's contributions prior to
review but will definitely do it after review with a double role:

* `cph` of `sf` code :sparkles:

* `rev` because of the improvements of the package thanks to his review :ok_hand:
