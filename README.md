# p5.wasm

An experimental addon library for p5.js written in Rust and compiled to WebAssembly. Reimplements common functions found in p5.js.

## Usage
The easiest way to use p5.wasm is to use the CDN link.
```html
<script src="https://cdn.jsdelivr.net/npm/p5.wasm@0.1.1/dist/p5.wasm.js"></script>
```

However if for any reason you cannot use the CDN link, you can continue reading, otherwise skip to the next two paragraphs.

Unlike most other addon libraries, you will need three files in order to use this addon library. This is due to how WebAssembly is designed but we won't go into details here. To note is that the release zip file will contain three files:

1. p5.wasm.js
2. index.bundle.js
3. p5.wasm

To use the files, you must put them in a folder called `wasm` in your web server's root, then in your HTML file after including `p5.js`, you can then include `p5.wasm.js` with a `<script>` tag. You should not include the other two files within the HTML file, they will be loaded by `p5.wasm.js`. If you cannot put the files in a folder called `wasm` at root, then you will need to follow the build steps outlined below to build a copy yourself with the right path.

It is not enough however to just include the files. To be able to start using p5.wasm you will need to wait for `index.bundle.js` and `p5.wasm` to finish loading asynchronously by `p5.wasm.js`, otherwise your p5 sketch may be initialized before p5.wasm's functions are attached to the p5 prototype object.

Fortunately a global promise attached to the `window` object is provided to handle this called `p5WasmReady`. Usage differs a bit depending on whether you are using global mode or instance mode.

### Global mode
```javascript
// This is to stop global mode from starting automatically
p5.instance = true;

// Write your p5 sketch as usual
function setup(){
	//...
}

function draw(){
	//...
}

// Wait for promise to resolve then start p5 sketch
window.p5WasmReady.then(() => {
	new p5();
});
```

### Instance mode
```javascript
// Write your p5 sketch as usual
const sketch = function(p){
	p.setup = function(){
		//...
	};

	p.draw = function(){
		//...
	}
};

// Wait for promise to resolve then start p5 sketch
window.p5WasmReady.then(() => {
	new p5(sketch);
});
```

It may be possible to skip waiting for the promise in global mode but support for it has not been implemented in p5.js yet. For now just use the provided `p5WasmReady` promise to get around it.

## API
Currently the following functions are implemented:

| p5.wasm                               | p5                             |
| ------------------------------------- |--------------------------------|
| `wasm.abs(n)`                         | `abs(n)`                       |
| `wasm.ceil(n)`                        | `ceil(n)`                      |
| `wasm.constrain(n, low, high)`        | `constrain(n, low, high)`      |
| `wasm.dist(x1, y1, x2, y2)`           | `dist(x1, y1, x2, y2)`         |
| `wasm.dist3d(x1, y1, z1, x2, y2, z2)` | `dist(x1, y1, z1, x2, y2, z2)` |
| `wasm.exp(n)`                         | `exp(n)`                       |
| `wasm.floor(n)`                       | `floor(n)`                     |
| `wasm.lerp(start, stop, amt)`         | `lerp(start, stop, amt)`       |
| `wasm.log(n)`                         | `log(n)`                       |
| `wasm.mag(x, y)`                      | `mag(x, y)`                    |
| `wasm.map(n, start1, stop1, start2, stop2, [withinBounds])` | `map(n, start1, stop1, start2, stop2, [withinBounds])` |
| `wasm.norm(n, start, stop)`           | `norm(n, start, stop)`         |
| `wasm.round(n)`                       | `round(n)`                     |
| `wasm.round_decimal(n, decimal)`      | `round(n, decimal)`            |
| `wasm.sq(n)`                          | `sq(n)`                        |
| `wasm.sqrt(n)`                        | `sqrt(n)`                      |
|                                       |                                |
| `wasm.noise(x)`                       | `noise(x)`                     |
| `wasm.noise2d(x, y)`                  | `noise(x, y)`                  |
| `wasm.noise3d(x, y, z)`               | `noise(x, y, z)`               |

Upcoming release:

| p5.wasm                               | p5                             |
| ------------------------------------- |--------------------------------|
| `wasm.fract(n)`                       | `fract(n)`                     |

## Development
To build p5.wasm, you will need Rust setup on your system, follow the steps outlined [here](https://rustwasm.github.io/docs/book/game-of-life/setup.html) to install `rustup`, `rustc`, `cargo` and `wasm-pack`, `cargo-generate` is not necessary unless you want to follow the tutorial in the Rust book (do try it, it's rather interesting). Also assuming you have node.js and npm already setup but if not, [download and install node.js](https://nodejs.org/).

After setting Rust up, clone this repo.

Next run `npm install`. It should build the included Rust project for you as well.

Once done, run `npm run build` to build the library. The three files mentioned above will be saved in the `dist` folder.

If you wish to change the path in which the files are supposed to be served from (default is `/wasm/`), you can run `ASSETS_PATH="your_path/" npm run build`, substituting `your_path` with where you wish to serve the file from. Note the trailing slash after `your_path`, it is required so that `p5.wasm` is requested from `your_path/p5.wasm` and not `your_pathp5.wasm`. You may want to include a leading slash as well to indicate absolute path, or even use a full web link starting with `https://`. Relative paths will be resolved from where the HTML that included `p5.wasm.js` is.

Whenever you made changes to either Rust or Javascript files, `npm run build` will take care of building both for you.

## Q&A
#### I need my code to run faster, will this addon library help?

**Short answer**: Probably not.

**Long answer**: Despite the hype, WebAssembly is not a magic bullet that will automatically make everything run faster. For most functions currently implemented, the native Javascript version still runs significantly faster in benchmark tests, especially those that are just aliases to Javascript standard library functions.

If you need your code to run faster, you should first identify where the bottle neck is, usually it is not a p5 function itself but rather the frequency in which you call it or the size of the calculation etc. You should start [here](https://github.com/processing/p5.js/wiki/Optimizing-p5.js-Code-for-Performance).

That said, there are potential speed gains to be had for example, `wasm.dist` and `wasm.mag` performs about 80% better than their Javascript counterparts and some other functions perform well when compared to Javascript implementation on Firefox but the performance gains are lost when running in Chrome.


#### When should I use this addon library?

I'm not sure actually. I create this little project over a Christmas break just as an excuse to use Rust for something.

I do aim to continue working on it and potentially also do some profiling to squeeze a bit more performance out of it, maybe some other p5.js functions really do benefit greatly in terms of speed when running on WebAssembly.

But until then, treat this as an experiment, which it is, and play with some Rust code if you are interested in helping develop this library!