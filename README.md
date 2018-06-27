# snipit
Catch band it logic.js
```javascript
/ global map variables
var markers = [];
var lati;
var longi;
var newVenue = "";
// var map;

var bandIs = function (){

  var name = window.location.href;
  var thing= name.split("/")
  var lastUrl= thing[thing.length-1];

    var bandQuery = lastUrl;

    var queryURL = "https://rest.bandsintown.com/artists/" + bandQuery + "/events?app_id=bandit";
    var queryURL2 = "https://rest.bandsintown.com/artists/" + bandQuery + "?app_id=bandit";

    $.ajax({
        url: queryURL2,
        method: "GET"
    }).then(function(resultsEvent){
      var image= resultsEvent.image_url;
      var name= resultsEvent.name;
      var newImage=$("<img>").attr({"src": image, "class": "img img-responsive img-fluid", "alt": "click here to go to"+resultsEvent.name+"'s facebook"});
      var newAncher=$("<a>").attr({"href": resultsEvent.facebook_page_url, "target": "_blank"});
      newAncher.append(newImage);
      var newName=$("<h1>").text(name);
      $("#artistImage").append(newAncher);
      $("#artistName").prepend(newName);
    });
    //  band is in town api
    $.ajax({
        url: queryURL,
        method: "GET"
    }).then(function(resultsEvent){

        // populate map with preset coordinates to the Anthem in DC
        initMapFirst();
        // looping through the array of upcoming events
      if (resultsEvent.length==0) {
          alert("There is no concert or event set up for band :"+bandQuery);
        }
        for ( var i =0; i<resultsEvent.length; i++){

            var rawDate=resultsEvent[i].datetime.split("T");
            var dateArr=[];
            dateArr.push(rawDate[0]);
            dateArr.push(" Time: ");
            var time=rawDate[1].split(":");
            if (time[0]>12){
              time[0]-=12;
              rawDate[1]=time.join(":");
              dateArr.push(rawDate[1]);
              dateArr.push("pm");
            }else{
              dateArr.push(rawDate[1]);
              dateArr.push("am");
            }

            var date = dateArr.join(" ");

        // looping through the array of upcoming events


            var venue = resultsEvent[i].venue;
            var lat = venue.latitude;
            var long = venue.longitude;
            var line = $("<hr>")
            var div = $("<div>").attr({
                "id": venue.name,
                "class": "event",
            });
            var ticket =$("<a class='btn bg-secondary getTix'>").text("GET Ticket");
              ticket.attr({"href": resultsEvent[i].url, "target":"_blank"});
            var showtime = $("<p class='date'>").text("Date: " + date);
            var city = $("<p class='city'>").text("City: " +venue.city);
            var name = $("<p class='venueButtons'>").text(venue.name).attr({
                "class": "venue",
                // Venue location information is set to the the data types below
                "data-venue": venue.name,
                "data-date": resultsEvent[i].datetime,
                "data-city": venue.city,
                "data-lat": lat,
                "data-long": long
            });
            if (i==0) {
              initMap(venue.name, venue.latitude, venue.longitude);
            };
            // render the information to the html page, this will be adjusted
            div.append(city, name, showtime, ticket, line);
            // this ajax call works but is currently being appended to a placeholder that does not exist
            $("#artistLocation").append(div);

        }
        yelpfunction();
        return(keys(bandQuery))
    })
}

// spotify functionality
// brings access tokens from server side to client and refreshes them
var keys = function(bandQuery){
  var spotifyQuery=bandQuery

  $("#dump").empty();
  $.get("/api", function(data){

      var refresh  = data[0].refresh_token;
  return refreshCall(refresh)
  })
  var refreshCall =function(refresh){
      $.ajax({
          url: "/refresh_token",
          data: {
              "refresh_token": refresh
          }
      }).done(function(data){
          access_token = data.access_token;
      return firstCall(access_token, spotifyQuery)
  })}
}
var firstCall = function(access_token, spotifyQuery){
  $.ajax({
  url: 'https://api.spotify.com/v1/search?q=' + spotifyQuery + '&type=Artist',
  beforeSend: function(xhr) {
       xhr.setRequestHeader("Authorization", "Bearer " + access_token)
  }, success: function(data){
      var id = data.artists.items[0].id;
      return topTracks(id, access_token);

  }
  })
}
var topTracks= function(id, access_token){
  $.ajax({
      url: "https://api.spotify.com/v1/artists/" + id + "/top-tracks?country=es",
      beforeSend: function(xhr) {
          xhr.setRequestHeader("Authorization", "Bearer " + access_token)
  }, success: function(data) {
      for (var i=0; i<3; i++){
          var topID = data.tracks[i].id;
          var div = $("<div>").attr("class", "tracks");
          var player = $("<iframe>").attr({
              "src": "https://open.spotify.com/embed/track/" + topID,
              "frameborder": 0,
              "allowtransparency": "true",
              "allow": "encrypted-media",
              "class": "topTracks"
          })
          div.append(player);
          $("#dump").append(div);
          // $("#dump").append(player);
      }
  }
  })
}
//var yelpfunction= function(){
  // function that occurs when the user pick a single venue

var yelpfunction=function(){
  $(".venue").on("click", function(){

    var newVenue = $(this).attr("data-venue")
    var lati = $(this).attr("data-lat")
    var longi = $(this).attr("data-long")
    var userSelects="bar, restaurant"; //have user select which catagory to select in html so they can choose what stores
    // clear yelp results on every click of venue
    $("#yelpResults").empty()
    initMap(newVenue, lati, longi)
    // addMarker(newVenue)

    var newSearchRequest= {
      categories: userSelects,
      latitude: lati,
      longitude: longi
    }
    $.post("/yelp", newSearchRequest, function(data){
      // console loging all the data as array and json object
      var location = [];
      // loop to go log all data in the array
      for (var i = 0; i < data.length; i++) {
        var labels = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ';
        var yelpResultsCard = $("<p>")
        var yelpImage = $("<img>")
        yelpImage.attr("src", data[i].img)
        yelpImage.attr("class", "card-img-top")
        var yelpName = data[i].name
        var add1 = data[i].address[0]
        var add2 = data[i].address[1]
        var add3 = data[i].address[2]
        var yelpPhone = data[i].phone
        var yelpPrice = data[i].price
        var yelpRating = data[i].rating
        $("#yelpResults").append('<a href='+data[i].yelp+' target="_blank"><div class="card text-center yelpCard">' +
          '<img class="card-image-top yelpImage" src="'+data[i].img+'">' +
          '<div class="card-body yelpInfo">' +
            '<h4 id="yelpName" class="card-title">' + yelpName +' ('+labels[i]+') </h4>' +
            '<p id="add1" class="card-text">' + add1 + '</p>' +
            '<p id="add2" class="card-text">' + add2 + '</p>' +
            // '<p id="add3" class="card-text">' + add3 + '</p>' +
            '<p id="yelpPhone" class="card-text">Phone: ' + yelpPhone + '</p>' +
            '<p id="yelpRating" class="card-text">Rating: ' + yelpRating + '</p>' +
            '<p id="yelpPrice" class="card-text">' + yelpPrice + '</p>' +
          '</div>' +
        '</div>' +
      '</div></a>')
        location.push(data[i].coordinates);

      }
      initMap(newVenue, lati, longi, location);
      // use this to get google map intergration and info we want to give out as output for all the store info
    });
  });
}


$("#submitBtn").on("click", function(event){
    event.preventDefault();
    if($("#bandName").val().trim() !== ""){
      var bandname = $("#bandName").val().trim();

      window.location.href = `/bands/${bandname}`;
    }
})

$("h1").on("click", function(event){
  event.preventDefault();
    window.location.href = `/`;
})
$(".logoImg").on("click", function(event){
  event.preventDefault();
    window.location.href = `/`;
})

bandIs();

// load map with preset venue first
function initMapFirst() {

  var theAnthem = {lat: 38.88848049, lng: -77.0302294} // replace capitalGrill with venue lati and longi
  var map = new google.maps.Map(document.getElementById('artistMap'), {
    zoom: 14, // zoom in to neighborboods near the venue
    center: theAnthem
  });
  var marker = new google.maps.Marker({
    position: theAnthem,
    map: map
    // icon: image,

  })
} // endo initMapFirst function
//* Google Maps JS *//
// get goole map function with marker set desired location
function initMap(newVenue, lati, longi, location) {


  // var lati = 38.88848049
  // var longi = -77.0302294
  newVenue = {lat: parseFloat(lati), lng: parseFloat(longi)} // replace capitalGrill with venue lati and longi
  var map = new google.maps.Map(document.getElementById('artistMap'), {
    zoom: 14, // zoom in to neighborboods near the venue
    center: newVenue
  });
  if (!location) {
    var musicIcon = "https://png.icons8.com/color/50/000000/musical.png";
    var marker = new google.maps.Marker({
      position: newVenue,
      map: map,
      icon: musicIcon
    });
  } else{
    for (var i = 0; i < location.length; i++) {
      location[i]
      var musicIcon = "https://png.icons8.com/color/50/000000/musical.png";
      var marker = new google.maps.Marker({
        position: newVenue,
        map: map,
        icon: musicIcon
      });

      for (i = 0; i < location.length; i++) {
        var labels = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ';

        marker = new google.maps.Marker({
          position: new google.maps.LatLng(parseFloat(location[i].latitude), parseFloat(location[i].longitude)),
          map: map,
          label: labels[i]
        });
    }
  }
}
    // icon: image,


}
initMap()

// Adds a marker to the map and push to the array.
      function addMarker(newVenue) {
        var map = new google.maps.Map(document.getElementById('artistMap'), {
          zoom: 10, // zoom in to neighborboods near the venue
          center: newVenue
        });
        var marker = new google.maps.Marker({
          position: newVenue,
          map: map
        });
        markers.push(marker);
      }


```
