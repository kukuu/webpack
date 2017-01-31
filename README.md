# Webpack - How the Dependency Graph works


Webpack is a Module bundler. It bundles a bunch of modules with require statements

Bundlers such as Browserify and Webpack are steadily taking over from Gulp and Grunt,
 and there is very little room left for both stream-based tasks. The majority of the work which you
  were previously doing with Gulp is now handled by Webpack — bundling and optimization for JavaScript,
   CSS and images; code splitting; targeted bundles for different environments (see isomorphic apps).

As a build tool Webpack puts all of your assets, including Javascript, images, fonts, and CSS, 
in a dependency graph. Webpack lets you use require() in your source code to point to local files, 
like images, and decide how they're processed in your final Javascript bundle, like replacing the path 
with a URL pointing to a CDN.


# When to use  Webpack.

If you're building a complex Front End application with many non-code static assets
 such as CSS, images, fonts, etc, then yes, Webpack will give you great benefits.

If your application is fairly small, and you don't have many static assets and you only need to 
build one Javascript file to serve to the client, then Webpack might be more overhead than you need.


# The Dependency Graph property
Webpack gives us a dependency graph. What does that mean?
In the early days, we "managed" Javascript dependencies by including files in a specific order:

<script src="jquery.min.js"></script>  
<script src="jquery.some.plugin.js"></script>  
<script src="main.js"></script>  
This was too slow because of excess HTTP requests. We graduated to concatenating and 
minifying our scripts
 in a build step:

// build-script.js
var scripts = [  
    'jquery.min.js',
    'jquery.some.plugin.js',
    'main.js'
].concat().uglify().writeTo('bundle.js');

// Everything our app needs!
<script src="bundle.js"></script>

This still relied on the order of concatenated files. Even worse, the code could only 
communicate through global variables! The first script declared a global jQuery variable, 
then jquery.some.plugin.js created a new global, or modified the global jQuery object. Yuck.
Now we use CommonJS or ES6 modules to put our Javascript in a true dependency graph. We make 
small files that explicitly describe what they need. 

// Version.js
module.exports = { version: 1.0 };  
// App.js

var config = require('./Version.js');  
console.log('App Version:', config.version); 

The browser doesn't support require(), so we use a build tool to transform the above files 
into a "bundled" file that the browser can execute properly.

What Does Webpack Actually Do?
Webpack lets you use require() on local "static assets," meaning non-code files.


Wait, you can't require() images in Javascript! What's going on? When you run Webpack, it searches through all of your code for require() calls. It compares the path string ../../assets/logo.png to the "loader" configuration you specify.


loaders: [  
    { test: /.png$/, loader: "file" }
]

In this example, when you require() file paths ending in .png (matching the above regular expression),
 Webpack sends that file to the file loader.
The file loader does two things. In the bundled Javascript code, it replaces the require() call with 
a URL string, making it valid Javascript. The string depends on how you configure Webpack. Maybe it becomes 
a CDN URL, like cdn.mysite.com/logo.png.

The file loader also spits out logo.png into some local folder you specify, like dist/. Now you simply upload
 the contents of dist/ to your CDN, deploy your new code, and the image is guaranteed to load on your site.
 Key concept: The require('logo.png') source code never actually gets executed in the browser (nor in Node.js).
  Webpack builds a new Javascript file, replacing require() calls with valid Javascript code, such as URLs. 
  The bundled file is what's executed by Node or the browser.

# What About Browserify, Grunt, Gulp?

Webpack puts your static assets (and source code) in a true dependency graph. Grunt and Gulp are only tools 
for working with files, and have no concept of a depdency graph.

Browserify is mainly a tool to transform require() calls that work in Node.js into calls that work in the browser.
 It's a dependency graph for your source code only. 


# The Good

Static assets in a dependency graph offers many benefits. Here's a few:
	•	Dead asset elimination. This is killer, especially for CSS rules. You only build 
	the images and CSS into your dist/ folder that your application actually needs.

	•	Easier code splitting. For example, because you know that your file Homepage.js only
	 requires specific CSS files, Webpack could easily build a homepage.css file to greatly 
	 reduce initial file size.

	•	You control how assets are processed. If an image is below a certain size, you could base64 
	encode it directly into your Javascript for fewer HTTP requests. If a JSON file is too big, you can
	 load it from a URL. You can require('./style.less') and it's automaticaly parsed by Less into vanilla CSS.

	•	Stable production deploys. You can't accidentally deploy code with images missing, or outdated styles.

	•	Webpack will slow you down at the start, but give you great speed benefits when used correctly. 
	You get hot page reloading. True CSS management. CDN cache busting because Webpack automatically 
	changes file names to hashes of the file contents, etc.
Webpack is the main build tool adopted by the React community. This makes finding help easier, 
and understanding Webpack more valuable.


# The not too Good

Webpack isn't perfect and has some pitfalls.
	•	The documentation is awful. I won't sugarcoat this. The language is often confusing, such as "webpack
	 takes modules with dependencies and generates static assets representing those modules." What? Even the 
	 page layout is problematic, with random sidebar entries you can't click on, and animated logos while 
	 you're trying to read.

	•	The source code is similarly painful.

	•	Configuring Webpack is a minefield for newcomers. The configuration file syntax is confusing. 
	It's best to look at established examples from any boilerplate project. Also check out the 
	new webpack-validator library.

	•	Webpack is maintained mostly by one person. The rapid community adoption and the thrust into the 
	spotlight means the ecosystem lags far behind the maturity of React. This has side effects, such as the poor quality of the documentation.

	•	Webpack introduces a nasty mini language in a 
	string:require("!style!css!less!bootstrap/less/bootstrap.less"); This syntax is almost never used, and barely explained, but it's all over the documentation. This string language is one of of Webpack's biggest design flaws in my opinion.

# What's The "Dev Server"?

Webpack comes with a built in "dev server"; a small express app for local development.
 You simply include one Javascript tag pointed to the server, like localhost:8080/assets/bundle.js,
  and get live code updating and asset management for free.

For large projects, Webpack isn't worth using without the dev server.

# Stop Programming With Globals

Traditional Front End™ programming relies mainly on global variables. CSS rules all exist in a global namespace. 
Applying CSS rules to elements relies on manually lining up the contents of global strings (selectors) correctly. 
A hard coded image path is a global, and you can't statically analyze your codebase to find outdated, moved, or deleted images. Using a custom font in your CSS means you globally defined that font file somewhere, and you better hope you loaded it at the right time!
Stop being a human compiler. Use a dependency graph.

# Using Vanilla JavaScript 

Still the idea of using JavaScript for build automation sounds quite reasonable, because it allows 

1. To reduce the stack of technologies used in the front-end project and/or project dependencies

2.  It’s truly cross-platform (as opposed to bash scripts for example)

3. There is avalanche of 3rd party libraries available to you via NPM registry which you can use directly
 without Gulp plugins inside your automation scripts.

This process is run in modern JavaScript browsers. Don’t forget to  add logging and error handling functionality.

The build.js script  in our example will be converted to ES5 on the fly and executed by Node.js. 
No need to have any global dependencies installed to make it work (other than Node.js and npm).
*/




