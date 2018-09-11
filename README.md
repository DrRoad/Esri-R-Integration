# Integrating Modeling and Forecasting with R

This repository provides an example of how a web-based Esri frontend can be integrated with the R programming language. This allows combining spatial data and analytics from the Esri ecosystem with the statistical programming and modeling capabilities available within R.

## Live Links
Demo Frontend: https://retailscientifics.github.io/Esri-R-Integration/

Training a Model: https://retailscientifics.github.io/Esri-R-Integration/Server/training.html

Standalone Test: https://retailscientifics.github.io/Esri-R-Integration/Server/standalone_test.html

## Key Files
On the client side, the key production files will be:
- config.json
	- Shows how to add a custom widget to WebappBuilder
- index.html
	- Includes modifications for pulling in external javascript libraries such as jQuery
- Widgets/RIntegration/Widget.html
	- The HTML template for the custom widget, including an input form
- Widgets/RIntegration/Widget.js
	- The code/logic for pushing the form to a server-side R API to obtain a prediction

And on the server side, the key production files are
- Server/api.R
	- A "productionalized" version of model code, which describes an API endpoint that receives new data, adjoins new variables (including spatial data from a shapefile), and runs the model to obtain a prediction
- Server/esri_demo.R
	- The main production file that is run. Sets up the server, sources all of the appropriate libraries and static files, and serves up the endpoint defined in api.R


To help demonstrate how these files came to be and how they function, the following notebooks have also been included:
- Server/training.Rmd
	- Shows how the pre-production step of building and testing a model can be carried out.
- Server/standalone_test.Rmd
	- Step-by-step mockups of how the server-side R code will receive new data and run the model on it.

## Workflow
The workflow demonstrated in this repository goes something like this:

- Build a model locally within R and export it as a file (Server/training.Rmd)
	- Generally predicts a response variable from a number of regressors
	- Regressors can include spatial data sourced from Esri shapefiles
- Test model locally on mockup input data (Server/standalone_test.Rmd)
- Bundle prediction code into a standalone function (Server/api.R)
- Host prediction code as an API endpoint on a server using the Plumber library (Server/esri_demo.R)
- Create a web map frontend with a form to supply new input data/regressors (Widgets/RIntegration/Widget.html)
- Point form at API endpoint to return new predictions (Widgets/RIntegration/Widget.js)


This repository can be downloaded and used as-is; the following describes how to reproduce this repository from scratch.

# Frontend Creation

## Creating a Blank Web AppBuilder App
*Note: you can skip this step by simply downloading this repository*
- If you don't have it already, install [node.js](https://nodejs.org/en/download/)
- Download the latest version of [Web AppBuilder for ArcGIS](https://developers.arcgis.com/web-appbuilder/)
- Unzip and open a terminal in the resulting directory
- `cd server`
- `npm install`
- `npm run start`
- Open a browser to the page that is served up - in this case, `localhost:3346`.
- Enter the URL for your ArcGIS Online Portal (in our case, `http://scientifics.maps.arcgis.com/`)
- It will now ask for an App ID - click "Help" and follow the instructions laid out there. For the URL, use `http://localhost:3346/webappbuilder`.
- When it is created, go to "Settings -> App Registration" to register an ID, using the same URL as the redirect.
	- Alternatively: Back on your localhost page, create a default 2D web map and save it. Navigate back to the main page, find the map you just made, and save it.
	- Unzip it to a directory.

## Run the Frontend App
- Option 1: Simply copy all of these files to a server somewhere and host them online to access.
- Option 2: Start a local https server:
	- First generate a self-signed SSL certificate to enable HTTPS:
	`openssl req -new -x509 -keyout server.pem -out server.pem -days 365 -nodes`
	- Run `python3 server.py` (note: this does require python 3) and navigate to `https://localhost:8000/`
- You may get an "unsafe connection", this can be bypassed for testing purposes. You should now be able to see a basic webmap: ![basic](images/2018/07/basic.png)

## Add a Custom Widget
- Install the widget generator with `npm install -g yo generator-esri-appbuilder-js`
- Make a new directory for your project and run `yo esri-appbuilder-js` to generate some initial files and set some settings.
- Run `yo esri-appbuilder-js:widget` to create your actual widget in the `./widgets` directory. This will provide the basic scaffolding around which a full widget can be built.
- Copy your new widget back into your app's widget directory. For us, this will be under `widgets/RIntegration`

# Backend Creation

- Install R, by either following the instructions at one of the [websites](https://cran.r-project.org/) or
	- On unix operating systems, running something equivalent to `sudo apt-get install r-base` suffices.
- (Optional) Install [RStudio](https://www.rstudio.com/products/rstudio/download/).
- Install the [plumber](https://www.rplumber.io/) package:
	- Launch R from the command line by typing `R` (or launch RStudio), and at the prompt, install  *plumber* package by typing `install.packages('plumber')`.
- Now you can set up a test endpoint. Here is an example:
	```R
	# In a file such as 'test.R'

	#* @get /sum
	function(a, b) {
		return(a + b)
	}

	# In a separate file such as 'server.R' or in the R command prompt
	library(plumber)
	r <- plumb('test.R') 	# The file made above
	r$run(port=8000) 	# Any free port
	```
	After entering these commands or sourcing `server.R`, you can now visit something like [http://localhost:8000/sum?a=1&b=2](http://localhost:8000/sum?a=1&b=2) in your browser to test the results.
