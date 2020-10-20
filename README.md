# Graphing an Intersection Point using Chart.js  
[Description](#description)  
[Intersection](#intersection)  
[Ratio](#ratio)  
[Setting Up](#setting-up)  
  
## Description  
The code found in `src/Chart.vue` can be used to graph an intersection point of two lines. It uses [Chart.js](https://www.chartjs.org/) to display the data. The math used to plot the intersection point is based off of ratios (simple coordinate system transformation). Below the process for finding and plotting the intersection point is described.  
**ISSUE** The subtitle is configured before xTickMax is set, so in order for the subtitle to display correctly, the x2 - x1 value is hard-coded.  
  
## Intersection  
The intersection is found by finding the slope of the two lines, then setting them equal to each other. First we need to find the slope of the two points.  
```
(y2 - y1) / (x2 - x1)  
last_point_y - first_point_y / last_point_x - first_point_x  
```
Each line will have its own slope and now they can be set equal.  
```
y1 = mx + b, y2 = nx + a  
mx + b = nx + a => (b - a) / (mx - nx) = x  
```
That yields the x intersection. The y intersection can be found by plugging in x in to the linear equation.  
```
y = m(x_intersection) + b (b is the y intersect when x = 0)  
```
Now we have the x and y intersects!  
  
## Ratio  
```  
16000|                      100|
     |                         |
     |    .(1.2, 8100)         |   .(pixel_x, pixel_y)
     |                         |
    0|_ _ _ _ _ _ _ _       369|_ _ _ _ _ _ _ _ 
      0              3         55              888
```
The trick is to find the ratio of 1.2 and 8100 to their respective chart and then use that ratio to plot the new point.   
```
chart_x subtract chart_x_min divided by chart_x_max subtract chart_x_min  
1.2 - 0 / 3 - 0 = x_scale  
Because the scales are going in opposite directions, we need to "flip" the chart y-axis by subtracting y from the chart_y_max.  
(chart_y_max subtract chart_y) - chart_y_min divided by chart_y_max subtract chat_y_min  
(16000 - 8100) - 0 / 16000 - 0 = y_scale  
```
The next step is to times the chart ratio by the pixel system:  
```
pixel_x_max subtract pixel_x_min times by x_scale  
(888-55) * x_scale = pixel_x  
pixel_y_max subtract pixel_y_min times by y_scale   
(369-100) * y_scale = pixel_y  
```
Now just add the x and y minimums to get the new point values:  
```
pixel_x += 55  
pixel_y += 100  
```
The intersection point is placed by using `style.left` and `style.top`. The pixel_x_max and pixel_x_min are found using the canvas object from Chart.js. The chartArea attribute is used to calculate the pixel coordinates and also provides the buffer to add to the point on the last step.  
  
## Setting Up  
Install [Node.js](https://nodejs.org/en/) and then run `npm install`. Once everything is installed `npm run serve` can be used to start the service. Navigate to `http://localhost:8080` to see the chart.  