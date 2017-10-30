+++
title = "Bloom Filter"
date = 2017-10-29T23:06:30-04:00
description = "Bloom Filters are probabilistic data structures that are useful for when you need to optimize costly remote calls"
draft = false
d3enabled=true
nomath=true
+++

## Bloom Filter visualization

<p><em>In the visualization below the circles represent the currently active check-result, and the squares represents the bit-array within the bloom-filter</em></p>
<div id="bloom-filter"></div>
<div id="bloom-filter-inputs">
	Add value to bloom-filter: <input type="text" id="bloom-add"><button class="btn btn-default" style="padding: 0px 10px 0px 10px;" onClick="bloomAdd(this.previousSibling)">»</button><br>
	Check bloom filter for value: <input type="text" id="bloom-check"><button class="btn btn-default" style="padding: 0px 10px 0px 10px;" onClick="bloomCheck(this.previousSibling)">»</button>
	<p id="bloom-filter-check-result"></p>
</div> 

<script>

function mod(n, m) {
	var rem = n % m;
	return (rem < 0) ? (rem + m) : rem;
}

function primeHashString(prime) {
	return function (str) {
		var hash = 0, i, chr;
		for (i = 0; i < str.length; i++) {
			chr   = str.charCodeAt(i);
			hash  = (hash * prime) + chr;
			hash |= 0; // Convert to 32bit integer
		}
		return hash;
	};
}

function newArray(size, val) {
	var arr = [];
	for (var i = size; i--; ) {
		arr.push(val);
	}
	return arr;
}

function BloomFilter(size, hashes) {
	if (this instanceof BloomFilter) {
		this.size = size;
		this.values = newArray(size, 0);
		this.hashes = hashes;
		this.dorefresh = () => null;
		return this;
	} else {
		return new BloomFilter(size, hashes);
	}
}

BloomFilter.prototype.add = function (e) {
	var self = this;
	self.hashes.forEach(h => {
		var putOne = mod(h(e), self.size);
		self.values[putOne] = 1;
	});
	this.dorefresh();
}

BloomFilter.prototype.contains = function (e) {
	var self = this;
	var idx = this.hashes.map(h => mod(h(e), self.size));
	var any = idx.map((i) => self.values[i] === 1);


	return {
		'result' : any.indexOf(false) === -1,
		'indexes' : idx
	}
}

BloomFilter.prototype.onrefresh = function (fn) {
	this.dorefresh = fn;
}


function bloomAdd(input) {}

function bloomCheck(input) {}


function drawBloom() {

	var bloom = BloomFilter(25, [
		primeHashString(31),
		primeHashString(17),
		primeHashString(11)
	]);

	var checkedIndexes = [];

	function render() {

		var width = document.getElementById('bloom-filter').clientWidth;
		var aspect = 8;
		var height = Math.floor(width / aspect);

		var block = width / bloom.size;
		var radius = block * 0.45;
		var offset = block * 0.05;

		d3.selectAll('#bloom-filter').selectAll('svg').remove();

		var svg = d3
			.select("#bloom-filter")
			.append('svg')
			.attr('width', width)
			.attr('height', height)
			.attr('perserveAspectRatio', 'xMinYMid');

		svg
			.selectAll('circle')
			.data(bloom.values)
			.enter()
			.append('circle')
			.attr('fill', (d, i) => {
				var isChecked = checkedIndexes.indexOf(i) !== -1;
				return isChecked ? 'green' : 'white';
			})
			.attr('stroke', 'black')
			.attr('cx', (d, i) => (2 * i + 1) * (radius + offset))
			.attr('cy', (d, i) => radius + offset)
			.attr('r', radius);

		svg
			.selectAll('rect')
			.data(bloom.values)
			.enter()
			.append('rect')
			.attr('fill', d => (d > 0) ? 'black' : 'white')
			.attr('stroke', 'black')
			.attr('x', (d, i) => (2 * i) * (radius + offset) + offset)
			.attr('y', (d, i) => block + offset)
			.attr('height', (d, i) => radius * 2)
			.attr('width', (d, i) => radius * 2);
	}

	window.addEventListener('resize', function () {
		render();
	});

	bloom.onrefresh(render);

	bloomAdd = function (input) {
		bloom.add(input.value);
		input.value = "";
	};

	bloomCheck = function (input) {
		var word = input.value;
		var contains = bloom.contains(word);
		checkedIndexes = contains.indexes;

		render();

		var p =document.getElementById('bloom-filter-check-result');
		p.textContent = "Check Result = (word=" + word + " / result=" + contains.result + ")" ;
		input.value = "";
	};

	render();
}

window.addEventListener('load', function () {
	drawBloom();
});
</script>
