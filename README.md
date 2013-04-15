# GPX Planet Tools

These tools are for processing planet.gpx, an enormous CSV file
consisting of point coordinates. [Get it here](http://planet.osm.org/gps/).

* `filter-points.pl`

	Extract points inside a polygon specified by a osmosis' poly file
	or a number of them (or just a bbox). Can be used to make a regional
	extracts of planet.gpx.

* `filter-gpx.pl`

    Make extracts for GPX planet dump. Accepts tar file on STDIN and
    produces tar file and a metadata header for each filtering polygon.

* `gpx2bitiles.pl`

	Renders planet.gpx or its part into "bitiles": bit arrays that can
	later be rendered into regular png images.

* `clean_bitiles.pl`

	Clean bitiles of salt-and-pepper type noise (there's a lot of it
        between latitudes -28.65 and 28.65).

* `bitiles2png.pl`

	Convert bitiles to png images and generate low zoom tiles from them.

* `checkstatus.pl`

    Draws an image of all existing tiles for a given zoom. Useful for checking
    rendering progress of `gpx2bitiles.pl` -- but only for CSV input.

You can create antialiased low zoom tiles using [this tool](https://github.com/AMDmi3/tiletool).

## EXAMPLES

1. Make a regional extract for Austria.

        xz -cdv gps-points.csv.xz | perl filter-points.pl -p austria.poly | xz > austria.csv.xz

    Result: `austria.csv.xz` with every point that lies inside austria.poly polygon.


2. Make regional extracts for Austria and Germany simultaneously.

    Create a file regions.lst with the following two lines:

        poly/europe/austria.poly
        poly/europe/germany.poly

    Run a script:

        xz -cdv gps-points.csv.xz | perl filter-points.pl -l regions.lst -o extracts -z

    Result: two files in `extracts` directory, `austria.csv.gz` and `germany.csv.gz`
    (gzipped because of `-z` switch).


3. Make regional extracts from GPX dump for Austria and Germany.

        xz -cdv gpx-planet-dump.tar.xz | perl filter-gpx.pl -l regions.lst -o extracts -z

        for f in extracts/*.gz ; do echo $f ; gzip -dc $f | cat ${f%.tar.gz}.metadata.tarc - \
        | xz > ${f%.gz}.xz ; rm $f ; rm ${f%.tar.gz}.metadata.tarc ; done


4. Create zoom 12 bitiles for Australia.

        xz -cdv gps-points.csv.xz | perl gpx2bitiles.pl -b 112,-44,154,-10 -o ausbitiles -z 12

    The process doesn't differ for GPX dump, except you have to untar it:

        tar -xJOf gpx-planet-dump.tar.xz | perl gpx2bitiles.pl -b 112,-44,154,-10 -o ausbitiles -z 12


5. The same, but in three processes.

    Run the following commands simultaneously (in different windows or processes):

        xz -cdv gps-points.csv.xz | perl gpx2bitiles.pl -b 112,-44,154,-10 -o ausbitiles -z 12 -t 1,3
        xz -cdv gps-points.csv.xz | perl gpx2bitiles.pl -b 112,-44,154,-10 -o ausbitiles -z 12 -t 2,3
        xz -cdv gps-points.csv.xz | perl gpx2bitiles.pl -b 112,-44,154,-10 -o ausbitiles -z 12 -t 3,3

    During its run, gpx2bitiles script creates temporary files `gpx2bitiles.state*`.
    Those files are needed in case something happens: processing a CSV file
    can take several days, and this is a mean to resume processing. If you have
    stopped the script and do not intend on resuming, do not forget to delete
    state files.


6. Remove noise from generated bitiles.

        perl clean_bitiles.pl -i ausbitiles -o acbitiles


7. Create PNG tiles for Australia, zoom levels 0 to 12.

        perl bitiles2png.pl -i acbitiles -o austiles -0 -v


8. Create PNG tiles for Sydney with light green dots, zooms 5 to 12.
   Write empty tiles where there are no points (JOSM requires them).

        perl bitiles2png.pl -i acbitiles -o sydneytiles -b 150.8,-34.1,151.4,-33.6 -z 12 -u 5 -c 0,255,0 -e -v


## AUTHOR

Written by Ilya Zverev. All scripts are under WTFPL license: you can
do whatever you want with them. Please do not make regional extracts
or tiles of your cat. Thanks.
