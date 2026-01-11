# HTRC Presentation

*October 28, 2020*

**Slide 17: Deriving Basic Illustration Metadata**

Hi, my name is Stephen Krewson! (2019 ACS grant participant)

- I'm (slowly) finishing up a PhD in the English dept at Yale University. I'm writing a literary history of experimental education movements in the early 19C United States (with special attention to developments in print media and the larger context of education reform and children's literature and psychology).
- Since late 2019, I've been working full time as a technical writer at Ab Initio Software in Massachusetts.

**Slide 18: 2,584,888**

By the numbers! Even just 50 years of data yields a LOT of illustrations. 19C is when volume counts get truly and increasingly large.

HTRC collaboration essential for working at this scale.

**Slide 19: Two-pass method for illustration extraction**

Before I show some results and discuss applications, a quick visualization of how a sample book is processed:

- How we identify candidate pages... (10-class model, trained on 5k candidate pages)
- How we extract... (2-class model, trained on 500 bounding boxes)
- Keep the classes "inline image" and "plate image"; throw away the junk

**Slide 20: Project outcome**

Takeaways

- The two-pass approach works! Public training datasets would be nice to have for improvement and transparency, however.
- New types of print media require new models. This would not work with color photography or the kind of spreads you see in 20C magazines. Much harder problem! So we can be thankful for the physical separation between the metal plate and the wood engraving block. It's the basis of the classifiers.

- We tried to vectorize all 2.5M crops with a third CNN model... the vectors are hard to work with without infrastructure. Also: Good low-dimensional representations are a moving target; best to prioritize the image data along with the essential contextual metadata.
- Use the right **visualization** tools for the job! PixPlot takes that basic metadata and raw image data and does something very useful with it (even identifying clusters). Furthermore, PixPlot continues to add different projections and ways of viewing image data. We will have a demo in a moment.
- **Scale**: A 50-year sample is not tractable for individuals or small teams. Ideally, HathiTrust would start to keep metadata about estimated image regions. But 50 years is great for then going in with more focused research questions
  - 50 years (1-2 TB) can reasonably be moved around on cloud VMs (though it's detail-oriented work and will cost some amount of money)
- Research ideas that emerged from the dataset: **Publishers** are a great unit of analysis, because the firm is really the site at which image assets are materially acquired and used. Printers and booksellers coordinate to produce multiple "classes" of books. Low prestige = reusing engravings from the existing stock. Munroe Francis of Boston provides an example.
- Comparison among publishers and across regions is the next step

**Slide 21: Identifying similar wood engravings using PixPlot**

This slide shows some motivation for the demo. The red squares have been added to mark some duplicate images. I was able to create this instance by getting the cropped images from the ACS project and adding some basic metadata and then ingesting it with the PixPlot tool, developed by Doug Duhaime at the Yale DH lab.

**Munroe Francis** was active in early 19C in Boston. Some high prestige books (Shakespeare edition). Some more job-like printing. City directories. Lots of didactic and moral works.

Now let's look at an example of copycat images discovered in this way.

**Slide 22: Side-by-side page images for "Ship on fire at sea"**

https://catalog.hathitrust.org/Record/001019612

Left: *Robinson Crusoe, with new engravings on wood* (late-1830s?)

- Alexander Anderson (1775-1870, American physician and engraver, active in New York) 
- Ship episode from *Farther adventures of Robinson Crusoe* (Daniel Defoe, 1719). Crusoe goes back to island with Friday and his nephew. Works to set up a kind of colonial order: https://en.wikipedia.org/wiki/The_Farther_Adventures_of_Robinson_Crusoe
- Literally hundreds of Crusoe adaptations and editions in century after its publication
- Thomas and John Bewick, the very influential British wood engravers, worked on an edition of Crusoe in 1789. See David Blewett (1995).

Right: *Paul Preston's Adventures* (1847)

- https://catalog.hathitrust.org/Record/011625127
- By Thomas Picton [Milner] (New York based, 1822-1891)
- Wrote pseudonymously for newspapers/magazines as Paul Preston and "A. Gothamite"
- Derivative of Crusoe, Goodrich's Peter Parley Series

**Slide 23: Text from duplicate ship at sea images**

Read the text. Matches up with previous slide.

**Slide 24: Picton alludes to his borrowing**

Conclusion: Paul Preston is a based around a "skeleton" of Alexander Anderson's Crusoe images.

**Slide 25: Demo**

http://oilpalm2.htrc.illinois.edu/krewson/output/#

Show these views:

- UMAP Grid
- By Year
- UMAP dim. reduction with jitter to prevent images occluding one another

Go to these clusters: 

- Cluster 15: example is just southeast of this cluster! Lots of landscape and ocean scenes.
- Cluster 4: Kings and queens
- Cluster 16: flourishes and paper discolorations (like "junk" classes)
- Cluster 3: lots of images of children, good amount of reuse

Open questions:

- What about color and differences in scanning process?
- What about duplicate editions of the same book? Apply basic dedup before doing this?
  - Well, sometimes those differences are interesting!
- Any way to compare against Internet Archive or IIIF?

**Slide 26: Acknowledgements**

This project was a very rewarding culmination of work I've been doing off and on since 2017. How to work with illustrations from digital libraries has long been a research interest, but having such a complete dataset is finally going to allow me to write something well-posed and interesting in my dissertation.

And beyond! I hope to write up the details of this data pipeline and would be happy to work collaboratively on this or similar datasets in the future.