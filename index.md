<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">

    <title>Tree Example</title>

    <style>
	
	.node {
		cursor: pointer;
	}

	.node circle {
	  fill: #fff;
	  stroke: steelblue;
	  stroke-width: 3px;
	}

	.node text {
	  font: 12px sans-serif;
	}

	.link {
	  fill: none;
	  stroke: #ccc;
	  stroke-width: 2px;
	}
	
    </style>

  </head>

  <body>

<!-- load the d3.js library -->	
<script src="http://d3js.org/d3.v3.min.js"></script>
	
<script>

var treeData=[
	{"name":"ALIS Parts","parent":"null","children":[
		{"name":"Beaufort MCAS","parent":"ALIS Parts", "children":[{"name":"VMFAT 501","parent":"Beaufort MCAS"}]},
		{"name": "Cherry Point Depot", "parent":"ALIS Parts","children":[{"name":"FRC East","parent":"Cherry Point Depot"}]},
		{"name":"CVN", "parent":"ALIS Parts","children":[{"name": "USS Lincoln","parent":"CVN"}]},
		{"name":"Edwards AFB","parent":"ALIS Parts", "children":[{"name":"17R Squadron", "parent":"Edwards AFB"},{"name":"USAF 31st TES,USMC VMX 22,USN VX-9","parent":"Edwards AFB"}]},
		{"name":"Eglin AFB","parent":"ALIS Parts", "children":[{"name":"58th Fighter Squadron", "parent":"Eglin AFB"},{"name":"VFA 101","parent":"Eglin AFB"}]},
		{"name":"Hill AFB","parent":"ALIS Parts", "children":[{"name":"4th Fighter Squadron", "parent":"Hill AFB"},{"name":"24th Fighter Squadron","parent":"Hill AFB"},{"name":"34th Fighter Squadron Deployment","parent":"Hill AFB"}]},
		{"name":"Hill AFB - Ogden Depot","parent":"ALIS Parts", "children":[{"name":"F-35 Prod Flight","parent":"Hill AFB - Ogden Depot"}]},
		{"name":"Iwakuni MCAS","parent":"ALIS Parts", "children":[{"name":"VMFA 121 GG3", "parent":"Iwakuni MCAS"},{"name":"VMFA 121 GGB","parent":"Iwakuni MCAS"}]},
		{"name":"LHD1","parent":"ALIS Parts", "children":[{"name":"LHD1","parent":"LHD1"}]},
		{"name":"Luke AFB","parent":"ALIS Parts", "children":[{"name":"56th Fighter Wing", "parent":"Luke AFB"},{"name":"56th Fighter Wing Deployment","parent":"Luke AFB"},{"name":"62nd Fighter Squadron","parent":"Luke AFB"},{"name":"63rd Fighter Squadron","parent":":Luke AFB"},{"name":"DET2","parent":"Luke AFB"}]},
		{"name":"NAS Lemoore","parent":"ALIS Parts", "children":[{"name":"VFA 125", "parent":"NAS Lemoore"},{"name":"VFA 147","parent":"NAS Lemoore"}]},
		{"name":"Nellis AFB","parent":"ALIS Parts", "children":[{"name":"422nd TES","parent":"Nellis AFB"}]},
		{"name":"Nellis AFB 16 WS","parent":"ALIS Parts", "children":[{"name":"16th Weapons Squadron","parent":"Nellis AFB 16 WS"}]},
		{"name":"Yuma MCAS","parent":"ALIS Parts", "children":[{"name":"MALS 13", "parent":"Yuma MCAS"},{"name":"VMFA 121 Deployment","parent":"Yuma MCAS"},{"name":"VMFA 211", "parent":"Yuma MCAS"}]}

	]}

]; 


// ************** Generate the tree diagram	 *****************
var margin = {top: 20, right: 120, bottom: 20, left: 120},
	width = 960 - margin.right - margin.left,
	height = 900 - margin.top - margin.bottom;
	
var i = 0,
	duration = 750,
	root;

var tree = d3.layout.tree()
	.size([height, width]);

var diagonal = d3.svg.diagonal()
	.projection(function(d) { return [d.y, d.x]; });

var svg = d3.select("body").append("svg")
	.attr("width", width + margin.right + margin.left)
	.attr("height", height + margin.top + margin.bottom)
  .append("g")
	.attr("transform", "translate(" + margin.left + "," + margin.top + ")");

root = treeData[0];
root.x0 = height / 2;
root.y0 = 0;
  
update(root);

d3.select(self.frameElement).style("height", "500px");

function update(source) {

  // Compute the new tree layout.
  var nodes = tree.nodes(root).reverse(),
	  links = tree.links(nodes);

  // Normalize for fixed-depth.
  nodes.forEach(function(d) { d.y = d.depth * 180; });

  // Update the nodes…
  var node = svg.selectAll("g.node")
	  .data(nodes, function(d) { return d.id || (d.id = ++i); });

  // Enter any new nodes at the parent's previous position.
  var nodeEnter = node.enter().append("g")
	  .attr("class", "node")
	  .attr("transform", function(d) { return "translate(" + source.y0 + "," + source.x0 + ")"; })
	  .on("click", click);

  nodeEnter.append("circle")
	  .attr("r", 1e-6)
	  .style("fill", function(d) { return d._children ? "lightsteelblue" : "#fff"; });

  nodeEnter.append("text")
	  .attr("x", function(d) { return d.children || d._children ? -13 : 13; })
	  .attr("dy", ".35em")
	  .attr("text-anchor", function(d) { return d.children || d._children ? "end" : "start"; })
	  .text(function(d) { return d.name; })
	  .style("fill-opacity", 1e-6);

  // Transition nodes to their new position.
  var nodeUpdate = node.transition()
	  .duration(duration)
	  .attr("transform", function(d) { return "translate(" + d.y + "," + d.x + ")"; });

  nodeUpdate.select("circle")
	  .attr("r", 10)
	  .style("fill", function(d) { return d._children ? "lightsteelblue" : "#fff"; });

  nodeUpdate.select("text")
	  .style("fill-opacity", 1);

  // Transition exiting nodes to the parent's new position.
  var nodeExit = node.exit().transition()
	  .duration(duration)
	  .attr("transform", function(d) { return "translate(" + source.y + "," + source.x + ")"; })
	  .remove();

  nodeExit.select("circle")
	  .attr("r", 1e-6);

  nodeExit.select("text")
	  .style("fill-opacity", 1e-6);

  // Update the links…
  var link = svg.selectAll("path.link")
	  .data(links, function(d) { return d.target.id; });

  // Enter any new links at the parent's previous position.
  link.enter().insert("path", "g")
	  .attr("class", "link")
	  .attr("d", function(d) {
		var o = {x: source.x0, y: source.y0};
		return diagonal({source: o, target: o});
	  });

  // Transition links to their new position.
  link.transition()
	  .duration(duration)
	  .attr("d", diagonal);

  // Transition exiting nodes to the parent's new position.
  link.exit().transition()
	  .duration(duration)
	  .attr("d", function(d) {
		var o = {x: source.x, y: source.y};
		return diagonal({source: o, target: o});
	  })
	  .remove();

  // Stash the old positions for transition.
  nodes.forEach(function(d) {
	d.x0 = d.x;
	d.y0 = d.y;
  });
}

// Toggle children on click.
function click(d) {
  if (d.children) {
	d._children = d.children;
	d.children = null;
  } else {
	d.children = d._children;
	d._children = null;
  }
  update(d);
}

</script>
	
  </body>
</html>
