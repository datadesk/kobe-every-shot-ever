# kobe-every-shot-ever
The code behind http://graphics.latimes.com/kobe-every-shot-ever/

This graphic uses data from stats.nba.com to plot all the shots Kobe Bryant ever took during NBA games (except two shots that are mysteriously missing). It uses Pandas, Matplotlib, CartoDB and Leaflet to scrape and display the data.

The Python code is in a Jupyter notebook in this repository. You can view it here on Github or run it yourself if you have Jupyter installed. To install all the necessary libraries:
	
	$ pip install -r requirements.txt

Once we scraped the data, we exported a csv and translated the x and y coordinates to latitude and longitude.

#####Learn from our mistakes

We did a very simple translation from X and Y coordinates to latitude and longitude. We opened the data in Excel and added two extra columns that added the X and Y values (divided by 1000) to the latitude and longitude of the Staples Center.

This solution is what we came up with on a deadline — and it worked. But next time, we'd do it differently. Here's why:

• One degree of latitude does not cover the same distance as one degree of longitude. So our court ended up slightly stretched vertically.

• Opening the data in Excel is a non-programmatic step that is hard to automate. It would have been a lot easier to just add the new columns in Pandas.

• It's fun to center the hoop on the Staples Center, but in our case it added nothing to the project. We should have just centered the map at 0,0 to make it easier to work with.


#####Get it in CartoDB
After adding lat and lon, we uploaded the data to CartoDB. Then it was time to display it in Leaflet. Here's the magic snippet:
	
	cartodb.createLayer(mapchart, {
	    id:"6eadec2c-0204-11e6-865f-0e674067d321",
	    user_name: 'latimes',
	    type: 'cartodb',
	    cartodb_logo: false,
	    sublayers: [{
	      sql: "SELECT * FROM kobe_all_shots_geocoded_final_merge ORDER BY priority,event_type DESC",
	      layer_name: "allshots",
	      cartocss: cssString,
	      interactivity: "combined_shot_type,action_type,season,event_type,shot_distance,playoffs,opponent,game_date",
	      attribution: "Source: stats.nba.com"
	    }]

	  });

cartodb.createLayer is a method of cartodb.js that lets you quickly create a leaflet layer from a CartoDB dataset. It uses CartoCSS for styling. Here's an example of that:

	var cssString = "#kobe_all_shots_geocoded_final_merge{\
	    marker-fill-opacity: 0.4;\
	    marker-line-color: #FFF;\
	    marker-line-width: 1;\
	    marker-line-opacity: 0;\
	    marker-placement: point;\
	    marker-type: ellipse;\
	    marker-width: 10;\
	    [event_type = 'Made Shot']{\
	        marker-fill: #6a3a89;\
	    }\
	      [event_type = 'Missed Shot']{\
	        marker-fill: #F5D05E;\
	        marker-line-width: 0.5;\
	        marker-fill-opacity: 0.5;\
	        marker-line-color: #cacaca;\
	        marker-line-opacity: 0.6\
	    }\
	    marker-allow-overlap: true;\
	     [zoom = 16] { \
	       marker-width: " + baseMarkerWidth * 24 +"; \
	     } \
	    [zoom = 15] { \
	       marker-width: " + baseMarkerWidth * 17 +"; \
	     } \
	    [zoom = 14] { \
	       marker-width: " + baseMarkerWidth * 10 +"; \
	     } \
	     [zoom = 13] { \
	       marker-width: " + baseMarkerWidth * 6 +"; \
	     } \
	     [zoom = 12] { \
	       marker-width: " + baseMarkerWidth * 4 + "; \
	     } \
	    [zoom = 11] { \
	       marker-width: " + baseMarkerWidth * 2 + "; \
	     } \
	    [zoom = 10] { \
	       marker-width: " + baseMarkerWidth + "; \
	     } \
	}";

