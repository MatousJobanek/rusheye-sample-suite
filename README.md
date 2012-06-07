Getting `rusheye.jar`
======================

Obtain the source from

    https://github.com/arquillian/arquillian-rusheye

and build it using

    cd arquillian-rusheye/
    mvn clean install
    
You can find the `rusheye.jar` necessary to run the test suite install

    arquillian-rusheye/rusheye-dist/target/rusheye.jar



Creating test suite
===================

Let's assume we have two directories, `screenshots1` and `screenshots2`.

The `screenshots1` is the directory with sample images (patterns),
while `screenshots2` is directory with new images which we would like
to compare with patterns.



Directory structure
-------------------

The crawl mechanism expects flat structure of crawled directory, e.g.:

    screenshots1/test1.png
    screenshots1/test2.png


Crawling directory
------------------

We can create a XML descritor of RushEye test suite by calling:

    java -jar rusheye.jar crawl screenshots1/ -O suite.xml
    
A file `suite.xml` with content similar to following one will be created:

    <?xml version="1.0" encoding="UTF-8"?>
    <visual-suite xmlns="http://www.jboss.org/rusheye/visual-suite" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://www.jboss.org/rusheye/visual-suite http://fryc.eu/rusheye/visual-suite.xsd">
    
      <global-configuration>
        <listener type="org.jboss.rusheye.parser.listener.CompareListener">
          ...
        </listener>
        <pattern-retriever type="org.jboss.rusheye.retriever.FileRetriever"/>
        <mask-retriever type="org.jboss.rusheye.retriever.FileRetriever"/>
        <sample-retriever type="org.jboss.rusheye.retriever.sample.FileSampleRetriever"/>
        <perception/>
      </global-configuration>
      
      <test name="test1">
        <pattern name="test1" source="test1.png"/>
      </test>
      
      ...


Running test suite
==================

Once we created `suite.xml` from `screenshots1` directory structure, we can use RushEye to run the test suite against images in `screenshots2` directory.


    java -jar rusheye.jar parse suite.xml  \
        -D result-output-file=result.xml  \
        -D samples-directory=screenshots2  \
        -D patterns-directory=screenshots1  \
        -D file-storage-directory=diffs

On the standard output, you can see following lines:

    [ DIFFER ] SeparatorTestCase.testThirdSeparator
    [ DIFFER ] SimpleEditorTestCase.testEditorSource
    [ DIFFER ] SpinnerTestCase.testLowerClickUp
      ...
    [ DIFFER ] TabPanelTestCase.testTabPanelExample
    [ DIFFER ] OrderingListTestCase.testUpMultipleSongsShift
    =====================
      Overall Statistics:
      DIFFER: 391
    =====================
        
The directory `diffs` will be created and an image comparing pattern with sample will be created for every test where pattern and sample image differs. This image highlights changes between the two compared images.


Adding mask to test suite
=========================

Since they are lot of differences in images, we can define masks to hide the unimportant differences.

There is currently only one type of mask:

* Selective-Alpha

There are predefined masks in `masks` directory:

* `masks/masks-selective-alpha`

Remove or clear the `diffs` directory

and run the crawl command again with `masks` directory specified and use `-f` to override existing `suite.xml`.

    java -jar rusheye.jar crawl screenshots1/ -O suite.xml -m masks/ -f
    
Now you can run the comparison script again:

    java -jar rusheye.jar parse suite.xml  \
        -D result-output-file=result.xml  \
        -D samples-directory=screenshots2  \
        -D patterns-directory=screenshots1  \
        -D file-storage-directory=diffs \
        -D masks-directory=masks


Now, you are getting much more stable results:

    [ SAME ] KeepAliveTestCase.testUsingIncorrectWay
    [ SAME ] PanelCustomizationTestCase.testThirdPanel
    [ SAME ] TooltipDataTableTestCase.testIterateThroughTable
    [ SAME ] ColorPickerTestCase.testPageSource
    [ SAME ] SimpleEditorTestCase.testStrikethroughButton
    [ SAME ] SeparatorTestCase.testFirstSeparator
    [ DIFFER ] TableSortingTestCase.testExternalSorting
    [ SAME ] TooltipTestCase.testDefaultToolTip
      ...
    [ SAME ] StyleTestCase.testBackgroundColor
    =====================
      Overall Statistics:
      SAME: 318
      DIFFER: 76
    =====================

