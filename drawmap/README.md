# Genetic Map Drawer
Finding good free software to plasmids seems to be difficult. I have no idea why. When trying to draw a chromosomal map of pSymB from _Sinorhizobium meliloti_, I developed this little AWK script.

To begin, download `drawmap`. On Linux, and MacOS, AWK is already available. On Windows, you can try [GAWK for Windows](http://gnuwin32.sourceforge.net/packages/gawk.htm); you'll have to, from the command line, type `gawk drawmap.awk yourfile > outputfile`. On Linux, MacOS, you can do a `chmod a+x drawmap` just once, then `./drawmap.awk yourfile > outputfile`.

You will have to describe the relevant features of your chromosome/plasmid in a plain text file. The first line, must be `size name` where the size is in nucleotides. You can then describe all the regions in your chromosome. The following are recognized:

<table>
<tr><th>Feature</th><th>Syntax</th></tr>
<tr><td>Transposon on the outside</td><td><c>t</c> name ntposition</td></tr>
<tr><td>Transposon on the inside</td><td><c>T</c> name ntposition</td></tr>
<tr><td>Genes on the inside</td><td><c>g</c> name ntposition</td></tr>
<tr><td>Genes on the outside</td><td><c>G</c> name ntposition</td></tr>
<tr><td>Region defined by nucleotides</td><td><c>r</c> name startnt stopnt ring</td></tr>
<tr><td>Region defined by transposons</td><td><c>rt</c> name starttp stoptp ring</td></tr>
<tr><td>Highlighted sector/wedge by nucleotides</td><td><c>s</c> startnt stopnt innerring outerring style</td></tr>
<tr><td>Highlighted sector/wedge by transposons</td><td><c>st</c> starttp stoptp innerring outerring style</td></tr>
<tr><td>Highlighted sector/wedge by one transposon and one nucleotide position</td><td><c>sT</c> starttp stopnt innerring outerring style</td></tr>
</table>

In the map generated, transposons were the features defining the regions, which were transposon-directed deletions; the name is somewhat misleading. The “transposons” are just features that have unique names between which you can define deletions. [Example map for pSymB](pSymB.pmap)

Lines starting with `#` are ignored. Also, you can highlight a wedge of the map using the `s` command. The formatting of the wedges is specified as an SVG style, with no spaces.

The default font is [Iwona](http://www.ctan.org/tex-archive/fonts/iwona/), which I quite like. If you edit the script, you can adjust the font easily in the `BEGIN` block, along with the radius of the plasmid, the various font sizes, the formatting of the ticks and region arcs and the formatting of the plasmid itself.
