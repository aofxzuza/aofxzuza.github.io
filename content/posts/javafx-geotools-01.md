+++
title = 'JavaFX with Geotools Tutorial - Part 1: Load shape file'
date = 2016-02-12T10:43:36+07:00
draft = false
tags = ['coding','application', 'javafx', 'geotools', 'java']
+++
![Tutorial 1](/imgs/geotools_fx/map_tutorial1.jpg "Tutorial 1")

## Prerequisites
*   [JDK 8u40](http://www.oracle.com/technetwork/java/javase/downloads/index.html) or greater (includes JavaFX 8)
*   A favorite text editor or IDE
*   [Maven](http://maven.apache.org/download.cgi)
*   [Geotools](http://www.geotools.org/)
*   [FXGraphics2D](https://github.com/jfree/fxgraphics2d)
*   Shape files from [DIVA-GIS](http://www.diva-gis.org/Data), [STATSilk](http://www.statsilk.com/maps/download-free-shapefile-maps) etc.

## Build with Maven
First you set up a basic build script.

### Create the project structure

1.  Create directory structure following maven standard
2.  Download [global country boundaries shape files](http://www.diva-gis.org/Data) and put to resource directory
3.  Create 2 classes for `Main.java` and `MapCanvas.java`

<pre>
map_example
└── src
    └── main
        └── java
            └── map_example
                └── Main.java
                └── MapCanvas.java
        └── resource
            └── map_example
                └── countries.dbf
                └── countries.prj
                └── countries.shp
                └── countries.shx
└── pom.xml
</pre>

### Configure pom.xml

1. Define the version number of libraries as you wish.
```xml
<properties>
	<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
	<geotools.version>15-SNAPSHOT</geotools.version>
	<fxgraphics2d.version>1.3</fxgraphics2d.version>
</properties>
```
+ Geotools
```xml
<dependency>
	<groupId>org.geotools</groupId>
	<artifactId>gt-shapefile</artifactId>
	<version>${geotools.version}</version>
</dependency>
<dependency>
	<groupId>org.geotools</groupId>
	<artifactId>gt-swing</artifactId>
	<version>${geotools.version}</version>
</dependency>
```
+ FXGraphics2D
```xml
<dependency>
	<groupId>org.jfree</groupId>
	<artifactId>fxgraphics2d</artifactId>
	<version>${fxgraphics2d.version}</version>
</dependency>
```

## Implementation
Now that you've done with setting up the project, next you can create map canvas.

### Create map canvas

1. Initial java fx canvas and `GraphicsContext`. The `GraphicsContext` will be used to draw map content on the canvas
```java
private Canvas canvas;
private MapContent map;
private GraphicsContext gc;

public MapCanvas(int width, int height) {
	canvas = new Canvas(width, height);
	gc = canvas.getGraphicsContext2D();
	initMap();
	drawMap(gc);
}

public Node getCanvas() {
	return canvas;
}
```
2. Load shape files and setting up style for displaying on the screen.
```java
private void initMap() {
	try {
		FileDataStore store = FileDataStoreFinder.getDataStore(this.getClass().getResource("countries.shp"));
		SimpleFeatureSource featureSource = store.getFeatureSource();
		map = new MapContent();
		map.setTitle("Quickstart");
		Style style = SLD.createSimpleStyle(featureSource.getSchema());
		FeatureLayer layer = new FeatureLayer(featureSource, style);
		map.addLayer(layer);
		map.getViewport().setScreenArea(new Rectangle((int) canvas.getWidth(), (int) canvas.getHeight()));
	} catch (IOException e) {
		e.printStackTrace();
	}
}
```
3.  Use StreamingRender to draw MapContent on the canvas

### Initial main application and screen size for display map
* `Main.java`
```java
public class Main extends Application {
	@Override
	public void start(Stage primaryStage) {
		try {
			MapCanvas canvas = new MapCanvas(1024, 768);
			Pane pane = new	Pane(canvas.getCanvas());
			Scene scene = new Scene(pane);
			primaryStage.setScene(scene);
			primaryStage.setTitle("Map Example");
			primaryStage.show();
		} catch (Exception e) {
				e.printStackTrace();
		}
	}

	public static void main(String[] args) {
		launch(args);
	}
}
```

## Source Code
[Github](https://github.com/aofxzuza/geotools_fx_tutorial)

## What's next?
In [Tutorial Part 2](/posts/javafx-geotools-02), we will add more functionality like pan, zoom in and zoom out.
