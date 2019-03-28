<html>
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="initial-scale=1,maximum-scale=1,user-scalable=no">
  <title>Tennessee Tornados</title>

  <link rel="stylesheet" href="https://js.arcgis.com/4.7/esri/css/main.css">
  <script src="https://js.arcgis.com/4.7/"></script>

  <style>
    html,
    body {
      padding: 0;
      margin: 0;
      height: 100%;
      width: 100%;
    }
	
	#viewDiv_2d {
		float: left;		
		height: 100%;
		width: 100%;
	}
	
	#optionsDiv {
	  position: absolute;
      top: 0px;
      right: 0px;
      max-width: 450px;
      background-color: dimgray;
      color: white;
	  text-align: center;
      z-index: 30;
      padding: 10px;
      border-radius: 5px;
    }
  </style>

  <script>
    require([
        "esri/views/MapView",
        "esri/views/SceneView", 
		"esri/WebMap",
		"esri/WebScene",
		"esri/widgets/Legend",
		"esri/layers/GraphicsLayer",
		"esri/symbols/SimpleMarkerSymbol",
		"esri/symbols/PointSymbol3D",
      	"esri/symbols/ObjectSymbol3DLayer",
		"esri/tasks/QueryTask",
		"esri/tasks/support/Query",
		"dojo/dom",
		"dojo/on",
		"dojo/_base/array",		
        "dojo/domReady!"
      ],
      function(
        MapView, SceneView, WebMap, WebScene, Legend, GraphicsLayer, SimpleMarkerSymbol, PointSymbol3D, ObjectSymbol3DLayer, QueryTask, Query, dom, on, arrayUtils
      ) {
	  
var view_2d;
		var results2DLyr = new GraphicsLayer(); 
		
create_2dView();
		
		on(dom.byId("doBtn"),"click", doQuery);
		
function create_2dView() {
			  var webmap = new WebMap({
				portalItem: {
				  id: "54e89c9d3b9a4242987115cafa4aa0fa"
				}
			  });
			  view_2d = new MapView({
				map: webmap,
				container: "viewDiv_2d"
			  });
			  
			
view_2d.when(function() {
				webmap.add(results2DLyr);
				var legend = new Legend({
					id: "legend_2d",
					view: view_2d
				})
				view_2d.ui.add(legend, "bottom-right");
				view_2d.watch("extent", function(response){
					if (response){
						view_2d.center = response.center;
					}
				});
				
view_2d.watch("scale", function(response){
					if (response){
						view_2d.scale = response;
					}
				});
				view_2d.watch("rotation", function(response){
					if (response){
						view_3d.goTo({
							heading: 0 - response
						});
					}
				});
				
			});				  
		}
function doQuery(){
			var featureLayerUrl = view_2d.map.layers.items[1].url + "/" + view_2d.map.layers.items[1].layerId;
			var qTask = new QueryTask({
		        url: featureLayerUrl
		    });
			var params = new Query({
	        	returnGeometry: true,
	        	outFields: ["*"]
	      	});
        	var expressionSign = dom.byId("signSelect");
      		var val = dom.byId("valInput").value;
        	params.where = "mag" + expressionSign.value + val;
	        qTask.execute(params)
	          .then(getResults)
	          .otherwise(promiseRejected);
		}
		
  function getResults(response) {
	        dom.byId("printResults").innerHTML = response.features.length + " result(s) found!";
			displayResultsIn2D(response);
	    }
	 function promiseRejected(err) {
	        console.error("Query failed: ", err.message);
	    }
		
function displayResultsIn2D(response) {
	      	results2DLyr.removeAll();
	        var featureResults2D = arrayUtils.map(response.features, function(feature) {
	          feature.symbol = new SimpleMarkerSymbol({
				  style: "line",
				  color: "green",
				  size: "8px",
				  outline: {
				    color: [ 0, 255, 0 ],
				    width: 6
				  }
				});
	          return feature;
	        });
			
results2DLyr.addMany(featureResults2D);
	        view_2d.goTo(featureResults2D);		  
		  }function displayResultsIn2D(response) {
        	results3DLyr.removeAll();
	        var featureResults2D = arrayUtils.map(response.features, function(feature) {
	          var newFeature = feature.clone();
	          newFeature.symbol = new PointSymbol2D({
	            symbolLayers: [new ObjectSymbolDLayer({
	              material: {
	                color: "green"
	              },
	              resource: {
	                primitive: "cone"
	              },
	              width: 300000,
	              height: 1000000
	            })]
	          });
	          return newFeature;
	        });
	        results2DLyr.addMany(featureResults2D);
		  }
		
      });
  </script>
</head>

<body>
	<div id="viewDiv_2d"></div>
	<div id="optionsDiv">
		Magnitude
		<select id="signSelect">
		  <option value=">">is greater than</option>
		  <option value="<">is less than</option>
		  <option value="=">is equal to</option>
		</select>
		<input id="valInput" value="2.5" />
		<button id="doBtn">Execute</button>
		<br>
		<p><span id="printResults"></span></p>
	</div>
</body>

</html>
