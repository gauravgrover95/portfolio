### Project Introduction


#### Description

[ccNetViz](http://helikarlab.github.io/ccNetViz/) is a high performance, lightweight, and customizable client-side library aimed to solve the problem of visualizing and analyzing complex network graphs on the web. It utilizes the power of parallelly computed web graphics technology, WebGL. Thus possess the potential to view tens of thousands of nodes.

The Library aims to break the efficiency limitation of visualizing large networks developed using sequentially computed SVG canvas graphics. This limitation can be generally observed in all popular web network visualization libraries.

My objective in the Google Summer of Code 2018 would be to extend ccNetViz to further improve its efficiency and implement the support for better data formatting.



### Work Done


#### Fix zoom for large networks

The library imposes a max-size limitation on the nodes to utilize maximum space available in the view. On zooming, the area under the mouse pointer must remain in focus for intuitive user experience. We proposed that the node under the mouse pointer should also stay in focus when zoomed. 

But we observed that due to the max-size limitation on the node, it was getting panned away. The issue mentioned in detail in this [Github issue](https://github.com/HelikarLab/ccNetViz/issues/7).

Commit history for resolution of this issue can be found [here](https://github.com/gauravgrover95/ccnetviz_demo/commits/master) and the code merged by my mentor Ales can be found [here](https://github.com/HelikarLab/ccNetViz/commit/2ec120f8379fbfee81ead1cdcaee0383cc4bb4d1).


#### Implement client side SDF sprite-sheet generation

Implementation of Signed Distance Fields (SDF) of commonly used text characters was performed by [Znbiz](https://github.com/Znbiz) in Google Summer of Code 2016. In the [project report](https://znbiz.github.io/gsoc2016/), the comparison of 3 available text drawing techniques and why SDF produces the highest quality text labels was discussed.

Limitations faced in the user experience with this technique were:
1. Dependency on the server
2. No real-time support for unique font processing

This year we accepted this as a challenge to eliminate the drawbacks.

A real-time client-side generation of SDF text was proposed. We researched for an efficient, linear time implementation of the distance transform algorithm developed at Brown University by *P. Felzenszwalb* and *D. Huttenlocher*. The algorithm was published in *Theory of Computing* journal in 2012 can be found [here](http://cs.brown.edu/people/pfelzens/dt/).

We also found an open-sourced port of this algorithm from C++ to JS performed by mapbox™ available [here](https://github.com/mapbox/tiny-sdf). Thanks to the author, [Vladimir Agafonkin](https://github.com/mourner) and other contributors for this work.

It was a challenge to optimize the code from mapbox™ library to fit in the current rendering pipeline of ccNetViz as it required the understanding of discrete mathematics, algorithms and image processing altogether. Code reading without existing documentations introduced another level of complexity.

Ultimately, the problem was solved and the JS module developed for this task can be found [here](https://github.com/HelikarLab/ccNetViz/blob/master/src/texts/sdf/spriteGenerator.js).


#### Test and debug client-side SDF sprite-sheet generation

Proper investment of time for testing and debugging was essential especially because of  the number of independent configuration variables mentioned below:

* Scale of graph
* Font-families
* Characters
* Font-sizes 
* Font-weight
* Italics
* Alignment

A number of artifacts for several reasons were observed. We found that the proper spacing in between characters produced majority of the issues. It was patched by developing another module that can be found [here](https://github.com/HelikarLab/ccNetViz/blob/master/src/texts/sdf/glyphTrimmer.js).

Proper tweaking of configuration for both the modules was performed for making the modules production ready. Complete commit history for this task can be found [here](https://github.com/HelikarLab/ccNetViz/pull/16).


#### Update the build-system

In the previous version, node-scripts were used to integrate webpack which loads the JS modules and google closure compiler which optimizes the compiled code.

In the current version, latest community accepted practice of using a single webpack.config.js file was implemented. [Webpack migration guide](https://webpack.js.org/migrate/4/) helped significantly to determine what will work and break during the upgrade. 

We attempted to merge webpack and [google-closure-compiler](https://github.com/google/closure-compiler-js) together but found that official webpack plugin did not support the latest webpack v4 ([GitHub issues](https://github.com/webpack-contrib/closure-webpack-plugin/issues/47)). 

So, we removed the dependency on closure compiler and chose [uglifyjs plugin](https://github.com/webpack-contrib/uglifyjs-webpack-plugin) instead.

The webpack build system and all the npm modules were updated. babel dependencies were one of the important npm package update. Immediate benefits were observed in the later stages of the project.

The corresponding commit history for the update process can be found [here](https://github.com/HelikarLab/ccNetViz/pull/16).


#### Compute layouts in the background thread

A significant work was done as part of GSoC 2017 by [Renato Fabbri](https://github.com/ttm) supporting several layouts for large graphs. This is one of the most important features of the library.

We observed that the layout computations were having different complexities and with increasing number of nodes/edges a significant time and computing resources were consumed. This caused blockage of main JS thread and as a result blockage of user interactivity. 

We proposed to perform [layout computation in the background](https://github.com/HelikarLab/ccNetViz/issues/9) using the multi-threading web API, web-workers.

Obstacles:

1. Webpack v4 was required for [worker-loader](https://github.com/webpack-contrib/worker-loader) plugin
2. Existing code architecture was written keeping default call-by-reference mechanism of JS objects in mind. While web-workers uses call-by-value mechanism by default.

It was also proposed to serialize and deserialize JS data objects to [Transferable Object](https://developer.mozilla.org/en-US/docs/Web/API/Transferable) as mentioned in the [HTML5 Rocks web-worker guide](https://www.html5rocks.com/en/tutorials/workers/basics/) for supporting call-by-reference mechanism. But that would’ve added another level of complexity in the code and was saved for later.

The task was achieved and the commit history can be found [here](https://github.com/HelikarLab/ccNetViz/pull/16) and [here](https://github.com/HelikarLab/ccNetViz/pull/22). The code uses latest [async-await](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function) syntax to write async code in a more intuitive and easy to understand manner as compared to the usage of plain JS promises.



#### Other fixes and contributions

- Proper error report for faulty layout name was produced. <[Github Issue](https://github.com/HelikarLab/ccNetViz/issues/21)> <[Code](https://github.com/HelikarLab/ccNetViz/pull/22/commits/e708bcdfb09b8b2da95084345b520bff2a561e11)>
- Some documentation was written alongside. <[here](https://github.com/HelikarLab/ccNetViz/commit/f2a377536f80d63395d6ea0fd078a572bd22762f#diff-331a629f1728de6fe30b04dbdd13568a)>, <[here](https://github.com/HelikarLab/ccNetViz/commit/ddf249a76293399a30cc7075d3a603866d97a68b#diff-88c0fa8951783d96e57d5221523465c9)>

Also, some issues and improvements for future builds were documented. Details can be found below:

- [https://github.com/HelikarLab/ccNetViz/issues/23](https://github.com/HelikarLab/ccNetViz/issues/23)
- [https://github.com/HelikarLab/ccNetViz/issues/26](https://github.com/HelikarLab/ccNetViz/issues/26)
- [https://github.com/HelikarLab/ccNetViz/issues/25](https://github.com/HelikarLab/ccNetViz/issues/25)
- [https://github.com/HelikarLab/ccNetViz/issues/24](https://github.com/HelikarLab/ccNetViz/issues/24)
- [https://github.com/HelikarLab/ccNetViz/issues/20](https://github.com/HelikarLab/ccNetViz/issues/20)
- [https://github.com/HelikarLab/ccNetViz/issues/8](https://github.com/HelikarLab/ccNetViz/issues/8)



### Future Works

In future, I believe the following tasks could further improve the library:

1. Multi-line label support
2. Label Alignment
3. Spatial Search for Edge Labels
4. Hiveplot algorithm can be improved by giving more choice to the user like classification of nodes and axis orientations.



### Conclusion

In summary, following features were completed:

- Fixed of the zoom feature
- Generated client-side Signed Distance Fields for labels on-demand
- Tested and optimized for Signed Distance Fields generation
- Updated plugins of the library: removal of closure compiler and installation uglifyJS
- Shifted build system from node-scripts to webpack cli build
- Installed of web-workers
- Computed layout in the background thread
- Fixed random layout value bug
- Wrote basic documentation for some files of the code
- Documented remaining low priority issues


### Acknowledgments

Many many thanks to my mentor and supervisor Dr. Tomas Helikar for considering me worthy enough to provide this golden learning and contributing opportunity.

My other awesomely experienced mentor Ales who helped me time to time and boosted the progress of the project.

Google Summer of Code program for this wonderful learning experience to put my first baby steps in the open-source world. I hope to keep contributing back to develop amazing tech.

All the open source contributors briefly mentioned above who freely distributed their work to advance the science and technology for the betterment of all.


Link to all commits: [https://github.com/HelikarLab/ccNetViz/commits/master?author=gauravgrover95](https://github.com/HelikarLab/ccNetViz/commits/master?author=gauravgrover95)



