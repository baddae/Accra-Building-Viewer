<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Accra Building Viewer</title>
    <link rel="stylesheet" href="https://js.arcgis.com/4.31/esri/themes/light/main.css">
    <style>
        html, body, #viewDiv {
            padding: 0;
            margin: 0;
            height: 100%;
            width: 100%;
        }

        /* Header styling */
        header {
            background-color: #0077ac; /* Header background color */
            color: white; /* Text color */
            padding: 1px 20px; /* Minimal padding (top/bottom: 4px, left/right: 20px) */
            text-align: left; /* Align content to the left */
            font-size: 18px; /* Font size */
            font-weight: bold; /* Bold text */
            box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2); /* Add a subtle shadow */
            z-index: 500; /* Lower z-index to avoid overlapping with map controls */
            width: 100%; /* Full width */
            position: fixed; /* Fix the header at the top */
            top: 0; /* Position at the top of the page */
            left: 0; /* Align to the left */
            display: flex; /* Use flexbox to align image and text */
            align-items: center; /* Vertically center the content */
            height: 40px; /* Set a fixed height for the header */
        }

        /* Style for the header image */
        .header-image {
            height: 30px; /* Set a fixed height for the image */
            width: auto; /* Maintain aspect ratio */
            margin-right: 10px; /* Add space between the image and text */
        }

        /* Remove margin from the <h1> element inside the header */
        header h1 {
            margin: 0; /* Remove default margin */
            padding: 0; /* Remove default padding */
            font-size: 30px; /* Match the header font size */
            line-height: 0.5; /* Reduce line height to minimize vertical space */
        }

        #searchDiv {
            position: absolute;
            top: 60px; /* Adjusted to account for the header */
            right: 20px; /* Position in the upper-right corner */
            z-index: 1000; /* Ensure the search widget is above the header */
            background: white;
            padding: 10px;
            border-radius: 5px;
            box-shadow: 0 1px 2px rgba(0, 0, 0, 0.3);
        }

        /* Add padding to the map container to avoid overlapping with the header */
        #viewDiv {
            padding-top: 40px; /* Match the height of the header */
        }

        /* Welcome message widget styling */
        .welcome-widget {
            position: fixed;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background: white;
            padding: 20px;
            border-radius: 10px;
            box-shadow: 0 4px 8px rgba(0, 0, 0, 0.3);
            z-index: 2000; /* Ensure it appears above everything else */
            max-width: 400px;
            text-align: center;
        }

        .welcome-content h2 {
            margin-top: 0;
            font-size: 20px;
            color: #0077ac;
        }

        .welcome-content p {
            font-size: 14px;
            line-height: 1.5;
            color: #333;
        }

        .welcome-content button {
            background-color: #0077ac;
            color: white;
            border: none;
            padding: 10px 20px;
            border-radius: 5px;
            cursor: pointer;
            font-size: 14px;
            margin-top: 15px;
        }

        .welcome-content button:hover {
            background-color: #005f8b;
        }

        .esri-feature {
            letter-spacing: 0em;
            line-height: 1.55rem;
            font-feature-settings: "liga" 1, "calt" 0;
            background: #fff;
            padding: 1em;
        }
    </style>
</head>
<body>
    <!-- Header -->
    <header>
        <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/5/59/Coat_of_arms_of_Ghana.svg/1280px-Coat_of_arms_of_Ghana.svg.png" alt="Coat of Arms of Ghana" class="header-image">
        <h1>Accra Building Viewer</h1>
    </header>

    <!-- Map container -->
    <div id="viewDiv"></div>

    <!-- Search widget -->
    <div id="searchDiv"></div>

    <!-- Welcome message widget -->
    <div id="welcomeWidget" class="welcome-widget">
        <div class="welcome-content">
            <h2>Welcome to the Interactive Accra Buildings Viewer</h2>
            <p>
                This map provides information on buildings in the Greater Accra Metropolitan region as well as their digital addresses.
                <br><br>
                <strong>Disclaimer:</strong> The data is synthetic, and the information provided is not accurate or a representation of the Ghana Post GPS address. However, a similar process was used to obtain the digital addresses for this region in the web app.
            </p>
            <button id="closeWelcome">Close</button>
        </div>
    </div>

    <!-- Scripts -->
    <script src="https://js.arcgis.com/4.31/"></script>
    <script>
        require([
            "esri/Map",
            "esri/views/MapView",
            "esri/layers/FeatureLayer",
            "esri/Graphic",
            "esri/layers/GraphicsLayer",
            "esri/widgets/Search",
            "esri/widgets/BasemapToggle",
            "esri/widgets/Home",
            "esri/core/promiseUtils",
            "esri/symbols/PictureMarkerSymbol" // Add this line
        ], function(
            Map, MapView, FeatureLayer, Graphic, GraphicsLayer, Search,
            BasemapToggle, Home, promiseUtils, PictureMarkerSymbol // Add PictureMarkerSymbol here
        ) {
            // Create the map
            const map = new Map({
                basemap: "gray-vector"
            });

            // Create the view
            const view = new MapView({
                container: "viewDiv",
                map: map,
                center: [-0.2, 5.6], // Center on Greater Accra, Ghana
                zoom: 12,
                popup: {
                    dockEnabled: true, // Dock the popup
                    dockOptions: {
                        buttonEnabled: false, // Disable the dock button
                        breakpoint: false // Prevent undocking
                    },
                    highlightEnabled: false // Disable popup highlighting
                },
                highlightOptions: {
                    color: [0, 0, 0, 0] // Fully transparent highlight color
                }
            });

            // Add the buildings layer
            const buildingsLayer = new FeatureLayer({
                url: "https://services.arcgis.com/E5vyYQKPMX5X3R3H/arcgis/rest/services/GAMA_Buildings_updated/FeatureServer",
                outFields: ["*"],
                renderer: {
                    type: "simple",
                    symbol: {
                        type: "simple-fill",
                        color: [0, 0, 0, 0], // No fill color
                        outline: {
                            color: [105, 105, 105], // Dark gray outline
                            width: 1
                        }
                    }
                },
                popupTemplate: {
                    title: "Building Information",
                    content: [
                        {
                            type: "fields",
                            fieldInfos: [
                                { fieldName: "Building_i", label: "Building ID" },
                                { fieldName: "owner", label: "Owner" },
                                { fieldName: "Year_Built", label: "Year Built" },
                                { fieldName: "Area", label: "Total Building Area" },
                                { fieldName: "Number_of_", label: "Number of Floors" },
                                { fieldName: "Building_v", label: "Building Value" },
                                { fieldName: "Land_Area", label: "Total Land Size" },
                                { fieldName: "Land_Title", label: "Land Title Status" },
                                { fieldName: "Valuation_", label: "Valuation Year" },
                                { fieldName: "Tax_Rate", label: "Tax Rate" },
                                { fieldName: "Annual_Pro", label: "Annual Property Tax" }
                            ]
                        }
                    ]
                }
            });
            map.add(buildingsLayer);

            // Add the digital addresses layer with transparency
            const digitalAddressLayer = new FeatureLayer({
                url: "https://services.arcgis.com/E5vyYQKPMX5X3R3H/arcgis/rest/services/GAMA_NEW_Digital_Addresses1/FeatureServer",
                outFields: ["*"],
                opacity: 0, // Make the layer fully transparent
                popupTemplate: {
                    title: "Digital Address",
                    content: [
                        {
                            type: "fields",
                            fieldInfos: [
                                { fieldName: "digital_ad", label: "Digital Address" }
                            ]
                        }
                    ]
                }
            });
            map.add(digitalAddressLayer);

            // Create a graphics layer for highlighting selected polygons with a bloom effect
            const highlightLayer = new GraphicsLayer({
                effect: "bloom(2.5, 0.5px, 0.15)" // Bloom effect
            });
            map.add(highlightLayer);

            // Create a graphics layer for the custom pin
            const pinLayer = new GraphicsLayer();
            map.add(pinLayer);

            // Configure the Search widget to use only the digitalAddressLayer
            const searchWidget = new Search({
                view: view,
                includeDefaultSources: false, // Disable the default ArcGIS World Geocoding Service
                sources: [{
                    layer: digitalAddressLayer,
                    searchFields: ["digital_ad"], // Field to search
                    displayField: "digital_ad", // Field to display in results
                    exactMatch: false, // Allow partial matches
                    outFields: ["*"], // Return all fields
                    name: "Digital Address", // Name of the search source
                    placeholder: "Search Digital Address", // Placeholder text
                    maxResults: 10, // Maximum number of results to display
                    maxSuggestions: 6, // Maximum number of suggestions to display
                    suggestionsEnabled: true, // Enable suggestions
                    zoomScale: 5000 // Zoom scale when a result is selected
                }]
            });

            // Set the custom result symbol for the search widget
            searchWidget.allSources.on("after-add", ({ item }) => {
                item.resultSymbol = new PictureMarkerSymbol({
                    url: "https://upload.wikimedia.org/wikipedia/commons/3/3b/Blackicon.png", // Image URL
                    width: "48px", // Increase the width
                    height: "48px" // Increase the height
                });
            });

            // Add the search widget to the top right corner of the view
            view.ui.add(searchWidget, "top-right");

            // Handle search result events
            searchWidget.on("search-complete", function(event) {
                // Clear previous pin graphics
                pinLayer.removeAll();

                // Get the search result
                const results = event.results;
                if (results && results.length > 0) {
                    const result = results[0];
                    const location = result.results[0].feature.geometry;

                    // Create a graphic for the custom pin image
                    const pinGraphic = new Graphic({
                        geometry: location,
                        symbol: new PictureMarkerSymbol({
                            url: "https://upload.wikimedia.org/wikipedia/commons/3/3b/Blackicon.png", // Image URL
                            width: "48px", // Increase the width
                            height: "48px" // Increase the height
                        })
                    });

                    // Add the pin graphic to the pin layer
                    pinLayer.add(pinGraphic);

                    // Zoom to the search result location
                    view.goTo({
                        target: location,
                        zoom: 16 // Adjust the zoom level as needed
                    });
                }
            });

            // Add a basemap toggle widget
            const basemapToggle = new BasemapToggle({
                view: view,
                nextBasemap: "satellite"
            });
            view.ui.add(basemapToggle, "bottom-left");

            // Add a home button
            const homeBtn = new Home({
                view: view
            });
            view.ui.add(homeBtn, "top-left");

            // Handle click events on the map
            view.on("click", function(event) {
                // Clear previous highlights
                highlightLayer.removeAll();

                // Query both layers for features at the clicked location
                view.hitTest(event).then(async function(response) {
                    // Filter out graphics from the digitalAddressLayer
                    const buildingGraphic = response.results.find(result => result.graphic.layer === buildingsLayer)?.graphic;

                    // Highlight only the buildings layer polygons (skip digitalAddressLayer)
                    if (buildingGraphic) {
                        const highlightGraphic = new Graphic({
                            geometry: buildingGraphic.geometry,
                            symbol: {
                                type: "simple-fill",
                                color: [255, 0, 0, 0.5], // Red fill with 50% opacity
                                outline: {
                                    color: [255, 0, 0], // Red outline
                                    width: 2
                                }
                            }
                        });
                        highlightLayer.add(highlightGraphic);
                    }

                    // Create an array to hold popup content
                    const popupContent = [];

                    // Add building information to the popup content
                    if (buildingGraphic) {
                        popupContent.push({
                            type: "fields",
                            title: "Building Information",
                            fieldInfos: [
                                { fieldName: "Building_i", label: "Building ID" },
                                { fieldName: "owner", label: "Owner" },
                                { fieldName: "Year_Built", label: "Year Built" },
                                { fieldName: "Area", label: "Total Building Area" },
                                { fieldName: "Number_of_", label: "Number of Floors" },
                                { fieldName: "Building_v", label: "Building Value" },
                                { fieldName: "Land_Area", label: "Total Land Size" },
                                { fieldName: "Land_Title", label: "Land Title Status" },
                                { fieldName: "Valuation_", label: "Valuation Year" },
                                { fieldName: "Tax_Rate", label: "Tax Rate" },
                                { fieldName: "Annual_Pro", label: "Annual Property Tax" }
                            ],
                            graphic: buildingGraphic
                        });
                    }

                    // Add digital address information to the popup content
                    const digitalAddressGraphic = response.results.find(result => result.graphic.layer === digitalAddressLayer)?.graphic;
                    if (digitalAddressGraphic) {
                        popupContent.push({
                            type: "fields",
                            title: "Digital Address",
                            fieldInfos: [
                                { fieldName: "digital_ad", label: "Digital Address" }
                            ],
                            graphic: digitalAddressGraphic
                        });
                    }

                    // Display the combined popup content
                    if (popupContent.length > 0) {
                        view.popup.open({
                            features: [buildingGraphic, digitalAddressGraphic].filter(Boolean), // Filter out null values
                            content: popupContent,
                            location: event.mapPoint
                        });
                    }
                });
            });

            // Show the welcome message when the app loads
            const welcomeWidget = document.getElementById("welcomeWidget");
            const closeWelcomeButton = document.getElementById("closeWelcome");

            // Close the welcome message when the button is clicked
            closeWelcomeButton.addEventListener("click", function() {
                welcomeWidget.style.display = "none";
            });
        });
    </script>
</body>
</html>
