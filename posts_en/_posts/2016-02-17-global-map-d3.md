---
layout: post
title: Interactive global map with D3.js
---

## Global map
Inspired by these posts ([this](http://techslides.com/demos/d3/worldmap-template.html), [this](http://bl.ocks.org/d3byex/0786bb8a413cac6a5f79) and [this](http://bl.ocks.org/patricksurry/5721459)), I create a spherical map as follow:

<!-- more -->

<div id='globalMap'>
<script src="http://d3js.org/d3.v3.min.js"></script>
<script src="http://d3js.org/topojson.v1.min.js"></script>
<script>
	var width=600,height=600
	var projection=d3.geo.orthographic()
		.scale(300)
		.clipAngle(90)
		.translate([width/2,height/2])

	var path=d3.geo.path()
		.projection(projection)

	var svg=d3.select('#globalMap')
		.append('svg')
		.attr({
			width:width,
			height:height,
			id:'sphericalMap'
		})
		.on("mousedown", mousedown)
	    .on("mousemove", mousemove)
	    .on("mouseup", mouseup)
	    .style({
            display:'block',
            margin:'0 auto',
            'background-color':'#dadaeb'     
        })

	var g=svg.append('g')

	var tooltip=d3.select('#globalMap')
		.append('div')
		.attr('class','tooltip')

	var url='https://raw.githubusercontent.com/altitudelabs/wanderawetest/master/app/map_data/world-topo-min.json';
	d3.json(url,function(error,world) {
		var countries=topojson.feature(world,world.objects.countries).features
		var neighbors=topojson.neighbors(world.objects.countries.geometries)
		var graticule=d3.geo.graticule()
		var color=d3.scale.category10()

		g.append('path')
			.datum(graticule)
			.attr({
				'd':path,
				class:'graticule'
			})
			.style({
				fill:'none',
				stroke:'#777',
				'stroke-width':'0.5px',
				'stroke-opacity':0.4
			})

		g.selectAll('.country')
			.data(countries)
			.enter()
			.append('path')
			.attr({
				d:path,
				fill:function(d,i) {
					return color(d.color=d3.max(neighbors[i],
						function (n) {return countries[n].color;})+1|0)
				},
				opacity:0.7
			})
			.on('mousemove',function(d,i) {
				var mouse=d3.mouse(svg.node()).map(function(d) {return parseInt(d)})

				d3.select(this).attr({
					'stroke-width':2,
					stroke:'white'
				})

				tooltip.attr('style','left:'+(mouse[0]+document.getElementById('sphericalMap').offsetLeft+10)+'px;top:'+(mouse[1]+document.getElementById('sphericalMap').offsetTop-15)+'px')
					.style({
						position:'absolute',
						background:'white',
						'font-size':'14px'
					})
					.html(d.properties.name)		
			})
			.on('mouseout',function(d) {
				d3.select(this).attr({
					'stroke-width':0,
					stroke:'none'
				})
				tooltip.style('display','none')
			})			
	})
	
	function trackballAngles(pt) {  	  
	  var r = projection.scale();
	  var c = projection.translate();
	  var x = pt[0] - c[0], y = - (pt[1] - c[1]), ss = x*x + y*y;

	  var z = r*r > 2 * ss ? Math.sqrt(r*r - ss) : r*r / 2 / Math.sqrt(ss);  

	  var lambda = Math.atan2(x, z) * 180 / Math.PI; 
	  var phi = Math.atan2(y, z) * 180 / Math.PI
	  return [lambda, phi];
	}
	
	function composedRotation(λ, ϕ, γ, δλ, δϕ) {
	    λ = Math.PI / 180 * λ;
	    ϕ = Math.PI / 180 * ϕ;
	    γ = Math.PI / 180 * γ;
	    δλ = Math.PI / 180 * δλ;
	    δϕ = Math.PI / 180 * δϕ;
	    
	    var sλ = Math.sin(λ), sϕ = Math.sin(ϕ), sγ = Math.sin(γ), 
	        sδλ = Math.sin(δλ), sδϕ = Math.sin(δϕ),
	        cλ = Math.cos(λ), cϕ = Math.cos(ϕ), cγ = Math.cos(γ), 
	        cδλ = Math.cos(δλ), cδϕ = Math.cos(δϕ);

	    var m00 = -sδλ * sλ * cϕ + (sγ * sλ * sϕ + cγ * cλ) * cδλ,
	            m01 = -sγ * cδλ * cϕ - sδλ * sϕ,
	                m02 = sδλ * cλ * cϕ - (sγ * sϕ * cλ - sλ * cγ) * cδλ,
	        m10 = - sδϕ * sλ * cδλ * cϕ - (sγ * sλ * sϕ + cγ * cλ) * sδλ * sδϕ - (sλ * sϕ * cγ - sγ * cλ) * cδϕ,
	            m11 = sδλ * sδϕ * sγ * cϕ - sδϕ * sϕ * cδλ + cδϕ * cγ * cϕ,
	                 m12 = sδϕ * cδλ * cλ * cϕ + (sγ * sϕ * cλ - sλ * cγ) * sδλ * sδϕ + (sϕ * cγ * cλ + sγ * sλ) * cδϕ,
	        m20 = - sλ * cδλ * cδϕ * cϕ - (sγ * sλ * sϕ + cγ * cλ) * sδλ * cδϕ + (sλ * sϕ * cγ - sγ * cλ) * sδϕ,
	            m21 = sδλ * sγ * cδϕ * cϕ - sδϕ * cγ * cϕ - sϕ * cδλ * cδϕ,
	                 m22 = cδλ * cδϕ * cλ * cϕ + (sγ * sϕ * cλ - sλ * cγ) * sδλ * cδϕ - (sϕ * cγ * cλ + sγ * sλ) * sδϕ;
	                 
	    if (m01 != 0 || m11 != 0) {
	         γ_ = Math.atan2(-m01, m11);
	         ϕ_ = Math.atan2(-m21, Math.sin(γ_) == 0 ? m11 / Math.cos(γ_) : - m01 / Math.sin(γ_));
	         λ_ = Math.atan2(-m20, m22);
	    } else {
	         γ_ = Math.atan2(m10, m00) - m21 * λ;
	         ϕ_ = - m21 * Math.PI / 2;
	         λ_ = λ;       
	    }
	    
	    return([λ_ * 180 / Math.PI, ϕ_ * 180 / Math.PI, γ_ * 180 / Math.PI]);
	}
	    
	var m0 = null,
	    o0;
	  
	function mousedown() {  
	  m0 = trackballAngles(d3.mouse(svg[0][0]));
	  o0 = projection.rotate();
	  d3.event.preventDefault();
	}

	function mousemove() {
	  if (m0) { 
	    var m1 = trackballAngles(d3.mouse(svg[0][0]));	    
	    o1 = composedRotation(o0[0], o0[1], o0[2], m1[0] - m0[0], m1[1] - m0[1])	   
	    projection.rotate(o1);	    	    
	    svg.selectAll("path").attr("d", path); 
	  }
	}

	function mouseup() {
	  if (m0) {
	    mousemove();
	    m0 = null;
	  }
	}
</script>
</div>


## Original codes
{% highlight javascript linenos %}
<script src="http://d3js.org/d3.v3.min.js"></script>
<script src="http://d3js.org/topojson.v1.min.js"></script>
<script>
	var width=600,height=600
	var projection=d3.geo.orthographic()
		.scale(300)
		.clipAngle(90)
		.translate([width/2,height/2])

	var path=d3.geo.path()
		.projection(projection)

	var svg=d3.select('#globalMap')
		.append('svg')
		.attr({
			width:width,
			height:height,
			id:'sphericalMap'
		})
		.on("mousedown", mousedown)
	    .on("mousemove", mousemove)
	    .on("mouseup", mouseup)
	    .style({
            display:'block',
            margin:'0 auto',
            'background-color':'#dadaeb'
        })

	var g=svg.append('g')

	var tooltip=d3.select('#globalMap')
		.append('div')
		.attr('class','tooltip')

	var url='https://raw.githubusercontent.com/altitudelabs/wanderawetest/master/app/map_data/world-topo-min.json';
	d3.json(url,function(error,world) {
		var countries=topojson.feature(world,world.objects.countries).features
		var neighbors=topojson.neighbors(world.objects.countries.geometries)
		var graticule=d3.geo.graticule()
		var color=d3.scale.category10()

		g.append('path')
			.datum(graticule)
			.attr({
				'd':path,
				class:'graticule'
			})
			.style({
				fill:'none',
				stroke:'#777',
				'stroke-width':'0.5px',
				'stroke-opacity':0.4
			})

		g.selectAll('.country')
			.data(countries)
			.enter()
			.append('path')
			.attr({
				d:path,
				fill:function(d,i) {
					return color(d.color=d3.max(neighbors[i],
						function (n) {return countries[n].color;})+1|0)
				},
				opacity:0.7
			})
			.on('mousemove',function(d,i) {
				var mouse=d3.mouse(svg.node()).map(function(d) {return parseInt(d)})

				d3.select(this).attr({
					'stroke-width':2,
					stroke:'white'
				})

				tooltip.attr('style','left:'+
					(mouse[0]+document.getElementById('sphericalMap').offsetLeft+10)+'px;top:'+
					(mouse[1]+document.getElementById('sphericalMap').offsetTop-15)+'px')
					.style({
						position:'absolute',
						background:'white',
						'font-size':'14px'
					})
					.html(d.properties.name)		
			})
			.on('mouseout',function(d) {
				d3.select(this).attr({
					'stroke-width':0,
					stroke:'none'
				})
				tooltip.style('display','none')
			})			
	})
	
	// Functions listed below for spherical rotation are copied from http://bl.ocks.org/patricksurry/5721459.
	function trackballAngles(pt) {  	  
	  var r = projection.scale();
	  var c = projection.translate();
	  var x = pt[0] - c[0], y = - (pt[1] - c[1]), ss = x*x + y*y;

	  var z = r*r > 2 * ss ? Math.sqrt(r*r - ss) : r*r / 2 / Math.sqrt(ss);  

	  var lambda = Math.atan2(x, z) * 180 / Math.PI; 
	  var phi = Math.atan2(y, z) * 180 / Math.PI
	  return [lambda, phi];
	}
	
	function composedRotation(λ, ϕ, γ, δλ, δϕ) {
	    λ = Math.PI / 180 * λ;
	    ϕ = Math.PI / 180 * ϕ;
	    γ = Math.PI / 180 * γ;
	    δλ = Math.PI / 180 * δλ;
	    δϕ = Math.PI / 180 * δϕ;
	    
	    var sλ = Math.sin(λ), sϕ = Math.sin(ϕ), sγ = Math.sin(γ), 
	        sδλ = Math.sin(δλ), sδϕ = Math.sin(δϕ),
	        cλ = Math.cos(λ), cϕ = Math.cos(ϕ), cγ = Math.cos(γ), 
	        cδλ = Math.cos(δλ), cδϕ = Math.cos(δϕ);

	    var m00 = -sδλ * sλ * cϕ + (sγ * sλ * sϕ + cγ * cλ) * cδλ,
	            m01 = -sγ * cδλ * cϕ - sδλ * sϕ,
	                m02 = sδλ * cλ * cϕ - (sγ * sϕ * cλ - sλ * cγ) * cδλ,
	        m10 = - sδϕ * sλ * cδλ * cϕ - (sγ * sλ * sϕ + cγ * cλ) * sδλ * sδϕ - (sλ * sϕ * cγ - sγ * cλ) * cδϕ,
	            m11 = sδλ * sδϕ * sγ * cϕ - sδϕ * sϕ * cδλ + cδϕ * cγ * cϕ,
	                 m12 = sδϕ * cδλ * cλ * cϕ + (sγ * sϕ * cλ - sλ * cγ) * sδλ * sδϕ + (sϕ * cγ * cλ + sγ * sλ) * cδϕ,
	        m20 = - sλ * cδλ * cδϕ * cϕ - (sγ * sλ * sϕ + cγ * cλ) * sδλ * cδϕ + (sλ * sϕ * cγ - sγ * cλ) * sδϕ,
	            m21 = sδλ * sγ * cδϕ * cϕ - sδϕ * cγ * cϕ - sϕ * cδλ * cδϕ,
	                 m22 = cδλ * cδϕ * cλ * cϕ + (sγ * sϕ * cλ - sλ * cγ) * sδλ * cδϕ - (sϕ * cγ * cλ + sγ * sλ) * sδϕ;
	                 
	    if (m01 != 0 || m11 != 0) {
	         γ_ = Math.atan2(-m01, m11);
	         ϕ_ = Math.atan2(-m21, Math.sin(γ_) == 0 ? m11 / Math.cos(γ_) : - m01 / Math.sin(γ_));
	         λ_ = Math.atan2(-m20, m22);
	    } else {
	         γ_ = Math.atan2(m10, m00) - m21 * λ;
	         ϕ_ = - m21 * Math.PI / 2;
	         λ_ = λ;       
	    }
	    
	    return([λ_ * 180 / Math.PI, ϕ_ * 180 / Math.PI, γ_ * 180 / Math.PI]);
	}
	    
	var m0 = null,
	    o0;
	  
	function mousedown() {  
	  m0 = trackballAngles(d3.mouse(svg[0][0]));
	  o0 = projection.rotate();
	  d3.event.preventDefault();
	}

	function mousemove() {
	  if (m0) { 
	    var m1 = trackballAngles(d3.mouse(svg[0][0]));	    
	    o1 = composedRotation(o0[0], o0[1], o0[2], m1[0] - m0[0], m1[1] - m0[1])	   
	    projection.rotate(o1);	    	    
	    svg.selectAll("path").attr("d", path); 
	  }
	}

	function mouseup() {
	  if (m0) {
	    mousemove();
	    m0 = null;
	  }
	}
</script>
{% endhighlight %}