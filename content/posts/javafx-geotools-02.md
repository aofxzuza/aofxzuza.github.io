+++
title = 'JavaFX with Geotools Tutorial – Part 2: Interacting with the User'
date = 2016-02-12T17:43:36+07:00
draft = false
tags = ['coding','application', 'javafx', 'geotools', 'java']
+++
![Tutorial 2](/imgs/geotools_fx/map_tutorial2.png "Tutorial 2")

## Interacting with the user.
Previously we can use `Geotools` to load a shape file, so now we will implement user interacting.

### Listening the mouse event

We will listen mouse event for
* drag map
* double clicked to restore to original state
* scroll to zoom in and out

#### Create a method `initEvent()`in `MapCanvas` class.
```java
private double baseDrageX;
private double baseDrageY;
/*
 * initial for mouse event
 */
private void initEvent() {
	/*
	 * setting the original coordinate
	 */
	canvas.addEventHandler(MouseEvent.MOUSE_PRESSED, new EventHandler<MouseEvent>() {

		@Override
		public void handle(MouseEvent e) {
			baseDrageX = e.getSceneX();
			baseDrageY = e.getSceneY();
			e.consume();
		}
	});
	/*
	 * translate according to the mouse drag
	 */
	canvas.addEventHandler(MouseEvent.MOUSE_DRAGGED, new EventHandler<MouseEvent>() {
		@Override
		public void handle(MouseEvent e) {
			double difX = e.getSceneX() - baseDrageX;
			double difY = e.getSceneY() - baseDrageY;
			baseDrageX = e.getSceneX();
			baseDrageY = e.getSceneY();
			DirectPosition2D newPos = new DirectPosition2D(difX, difY);
			DirectPosition2D result = new DirectPosition2D();
			map.getViewport().getScreenToWorld().transform(newPos, result);
			ReferencedEnvelope env = new ReferencedEnvelope(map.getViewport().getBounds());
			env.translate(env.getMinimum(0) - result.x, env.getMaximum(1) - result.y);
			doSetDisplayArea(env);
			e.consume();

		}
	});
	/*
	 * double clicks to restore to original map
	 */
	canvas.addEventHandler(MouseEvent.MOUSE_CLICKED, new EventHandler<MouseEvent>() {
		@Override
		public void handle(MouseEvent t) {
			if (t.getClickCount() > 1) {
				doSetDisplayArea(map.getMaxBounds());
			}
			t.consume();
		}
	});
	/*
	 * scroll for zoom in and out
	 */
	canvas.addEventHandler(ScrollEvent.SCROLL, new EventHandler<ScrollEvent>() {

		@Override
		public void handle(ScrollEvent e) {
			ReferencedEnvelope envelope = map.getViewport().getBounds();
			double percent = e.getDeltaY() / canvas.getWidth();
			double width = envelope.getWidth();
			double height = envelope.getHeight();
			double deltaW = width * percent;
			double deltaH = height * percent;
			envelope.expandBy(deltaW, deltaH);
			doSetDisplayArea(envelope);
			e.consume();
		}
	});
}

protected void doSetDisplayArea(ReferencedEnvelope envelope) {
	map.getViewport().setBounds(envelope);
	repaint = true;
}
```

## Painting improvement
We do not repaint every events cause it will overload CPU and making the application lagging.

#### Create `ScehduleService` to paint for 50.0 hz

```java
private static final double PAINT_HZ = 50.0;
private void initPaintThread() {
	ScheduledService<Boolean> svc = new ScheduledService<Boolean>() {
		protected Task<Boolean> createTask() {
			return new Task<Boolean>() {
				protected Boolean call() {
					Platform.runLater(() -> {
						drawMap(gc);
					});
					return true;
				}
			};
		}
	};
	svc.setPeriod(Duration.millis(1000.0 / PAINT_HZ));
	svc.start();
}
```
#### Update method `drawMap(GraphicsContext gc)` to redraw when event occur.

```java
private boolean repaint = true;
private void drawMap(GraphicsContext gc) {
	if (!repaint) {
		return;
	}
	repaint = false;
	StreamingRenderer draw = new StreamingRenderer();
	draw.setMapContent(map);
	FXGraphics2D graphics = new FXGraphics2D(gc);
	graphics.setBackground(java.awt.Color.WHITE);
	graphics.clearRect(0, 0, (int) canvas.getWidth(), (int) canvas.getHeight());
	Rectangle rectangle = new Rectangle((int) canvas.getWidth(), (int) canvas.getHeight());
	draw.paint(graphics, rectangle, map.getViewport().getBounds());
}
```

## Source Code
[Github](https://github.com/aofxzuza/geotools_fx_tutorial)
