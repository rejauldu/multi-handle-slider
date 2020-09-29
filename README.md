# Multi-Handle Slide

> A JavaScript plugin for two handale range slider.

## Description

A JavaScript, CSS and HTML plugin to create a range slider with two handle. This range slider is independent of any other plugin like JQuery or Bootstrap. This slider will be useful in filter page in e-commerce website's price range.

## Browser Support

We do care about it.

![IE](https://cloud.githubusercontent.com/assets/398893/3528325/20373e76-078e-11e4-8e3a-1cb86cf506f0.png) | ![Chrome](https://cloud.githubusercontent.com/assets/398893/3528328/23bc7bc4-078e-11e4-8752-ba2809bf5cce.png) | ![Firefox](https://cloud.githubusercontent.com/assets/398893/3528329/26283ab0-078e-11e4-84d4-db2cf1009953.png) | ![Opera](https://cloud.githubusercontent.com/assets/398893/3528330/27ec9fa8-078e-11e4-95cb-709fd11dac16.png) | ![Safari](https://cloud.githubusercontent.com/assets/398893/3528331/29df8618-078e-11e4-8e3e-ed8ac738693f.png)
--- | --- | --- | --- | --- |
IE 10+ ✔ | Latest ✔ | Latest ✔ | Latest ✔ | Latest ✔ |

## Getting started

Three quick start options are available:

* [Download latest release](https://github.com/rejauldu/multi-handle-slider/releases)
* Clone the repo: `git@github.com:rejauldu/multi-handle-slider.git`

## Usage

Create an element with the class `multi-handle-slider` with three span inside it as like:

```html
<div class="multi-handle-slider" data-min="0" data-max="100">
	<span class="handle-1"></span>
	<span class="highlight"></span>
	<span class="handle-2"></span>
</div>
```

Include Javascript:

```html
(function() {
	var slides = document.getElementsByClassName("multi-handle-slider");
	if(!slides.length)
		return false;
	var handle1Clicked = false;
	var handle2Clicked = false;
	
	var slide = slides[0];
	
	for(let i=0; i<slides.length; i++) {
		slides[i].setAttribute("data-handle-1-position", updateHandle1(slides[i]));
		slides[i].setAttribute("data-handle-2-position", updateHandle2(slides[i]));
		slides[i].querySelector(".handle-1").addEventListener('mousedown', e => {
			slide = e.target.parentElement || e.srcElement.parentElement;
			handle1Clicked = true;
		});
		slides[i].querySelector(".handle-2").addEventListener('mousedown', e => {
			slide = e.target.parentElement || e.srcElement.parentElement;
			handle2Clicked = true;
		});
	}
	document.addEventListener('mousemove', event => {
		event = getMouseEvent(event);
		var left = slide.getBoundingClientRect().left;
		
		var position = event.pageX-left-10;
		if(handle1Clicked) {
			updateHandle1(slide, position);
		} else if(handle2Clicked) {
			updateHandle2(slide, position);
		}
	});
	document.addEventListener('mouseup', e => {
		let updated = slide.getAttribute("data-updated");
		if(updated && (handle1Clicked || handle2Clicked)) {
			window[updated]();
		}
		handle1Clicked = false;
		handle2Clicked = false;
	});
})();
function getMouseEvent(event) {
	var eventDoc, doc, body;
	event = event || window.event;
	if (event.pageX == null && event.clientX != null) {
		eventDoc = (event.target && event.target.ownerDocument) || document;
		doc = eventDoc.documentElement;
		body = eventDoc.body;
		event.pageX = event.clientX + (doc && doc.scrollLeft || body && body.scrollLeft || 0) - (doc && doc.clientLeft || body && body.clientLeft || 0);
	}
	return event;
}
function updateHandle1(slide, position = null) {
	let width = slide.offsetWidth-20;
	let logarithm = slide.getAttribute("data-logarithm");
	let handle_1 = log(slide.getAttribute("data-handle-1"), logarithm);
	let handle1 = slide.querySelector(".handle-1");
	let highlight = slide.querySelector(".highlight");
	let min = log(slide.getAttribute("data-min"), logarithm);
	let max = log(slide.getAttribute("data-max"), logarithm);
	
	let onchange = slide.getAttribute("data-onchange");
	let handle2Position = slide.getAttribute("data-handle-2-position");
	let range = max-min;
	handle_1 = handle_1?handle_1:min;
	let unit = width/range;
	if(position == null)
		position = (handle_1 - min) * unit;
	position = position<0?0:position;
	let value = round(position/unit, logarithm);
	
	let handle1Position = position;
	let minimum = slide.querySelector(".minimum");
	handle1Position = handle1Position>width-handle2Position?width-handle2Position:handle1Position;
	handle1.style.transform = "translate("+handle1Position+"px, 0)";
	highlight.style.left = handle1Position+"px";
	highlight.style.width = width-handle1Position-handle2Position+"px";
	console.log(handle_1);
	if(minimum) {
		minimum.value = Math.round(alog(handle1Position/unit+min, logarithm));
	}
	if(onchange)
		window[onchange](Math.round(handle1Position/unit)+parseInt(min), range-Math.round(handle2Position/unit))+parseInt(min);
	slide.setAttribute("data-handle-1-position", handle1Position);
	return handle1Position;
}
function updateHandle2(slide, position = null) {
	let width = slide.offsetWidth-20;
	let logarithm = slide.getAttribute("data-logarithm");
	let handle_2 = log(slide.getAttribute("data-handle-2"), logarithm);
	let handle2 = slide.querySelector(".handle-2");
	let highlight = slide.querySelector(".highlight");
	let min = log(slide.getAttribute("data-min"), logarithm);
	let max = log(slide.getAttribute("data-max"), logarithm);
	
	let onchange = slide.getAttribute("data-onchange");
	let handle1Position = slide.getAttribute("data-handle-1-position");
	let range = max-min;
	handle_2 = handle_2?handle_2:max;
	let unit = width/range;
	if(position == null)
		position = (handle_2 - min) * unit;
	position = position>width?width:position;
	let value = round((width-position)/unit, logarithm); /* value from reverse side */
	
	let handle2Position = value*unit;
	let maximum = slide.querySelector(".maximum");
	handle2Position = handle2Position>width-handle1Position?width-handle1Position:handle2Position;
	handle2.style.transform = "translate(-"+handle2Position+"px, 0)";
	highlight.style.width = width-handle1Position-handle2Position+"px";
	
	if(maximum)
		maximum.value = Math.round(alog(range-handle2Position/unit+min, logarithm));
	if(onchange)
		window[onchange](Math.round(handle1Position/unit)+parseInt(min), range-Math.round(handle2Position/unit))+parseInt(min);
	slide.setAttribute("data-handle-2-position", handle2Position);
	return handle2Position;
}
function round(value, logarithm = false) {
	value = parseFloat(value);
	if(logarithm)
		return value;
	return Math.round(value);
}
function log(value, logarithm = false) {
	value = parseFloat(value);
	if(logarithm) {
		value = value?value:1;
		return Math.log(value);
	}
	return value;
}
function alog(value, logarithm = false) {
	value = parseFloat(value);
	if(logarithm)
		return Math.pow(Math.E, value);
	return value;
}
```

Include plugin's CSS:

```html
.multi-handle-slider {
	height:6px;
	background:#FD5;
	position:relative;
	z-index:1;
}
.handle-1, .handle-2 {
	position:absolute;
	top:-7px;
	width:18px;
	height:18px;
	background:#FFF;
	border: 1px solid #DDD;
	z-index:2;
	border-radius: 10px;
	box-shadow:0 0 3px #AAA;
	cursor:pointer;	
}
.handle-1:active, .handle-2:active {
	border: 1px solid #DCC;
	z-index:3;
	background:#FEE;
	box-shadow:0 0 0 #AAA;
}
.handle-1 {
	left:0;
}
.handle-2 {
	right:0;
}
.highlight {
	height:6px;
	background:#FA5;
	position:absolute;
	top:0;
	left:0;
	display:inline-block;
	width:100%;
}
```

And that's it.

[Check full example's source code](https://github.com/rejauldu/multi-handle-slider/blob/master/multi-handle-slider.html).

## Options

Here's a list of available settings.

```javascript
<div class="multi-handle-slider" data-min="0" data-max="100" data-handle-1="20" data-handle-2="80" data-onchange="cb" data-updated="updated" data-logarithm="true">
	<span class="handle-1"></span>
	<span class="highlight"></span>
	<span class="handle-2"></span>
	<input type="hidden" class="minimum" name="one" value="0" />
	<input type="hidden" class="maximum" name="two" value="100" />
</div>
```

Attribute		| Type			| Default		| Description
---				| ---			| ---			| ---
`data-min`		| *Integer*		| `Undefined`	| Minimum value for the range slider. Mandatory.
`data-max`		| *Integer*		| `Undefined`	| Maximum value for the range slider. Mandatory.
`data-handle-1`	| *Integer*		| `data-min`	| Initial position of the first handle. Optional.
`data-handle-2`	| *Integer*		| `data-max`	| Initial position of the second handle. Optional.
`data-onchange`	| *Function*	| `NULL`		| Callback function. This function is called dynamically when first or second handle is called. Optional.
`data-updated`	| *Function*	| `NULL`		| Callback function. This function is called after first or second handle is called. Optional.
`data-logarithm`| *Boolean*		| `false`		| Determines if this range slider's output value change exponentially or not. Optional.


## History

Check [Releases](https://github.com/rejauldu/multi-handle-slider/releases) for detailed changelog.

## Credits

All credit goes to [OnBiponi](https://onbiponi.com).

## License

[MIT License](https://rejauldu.mit-license.org/) © Rejaul Karim