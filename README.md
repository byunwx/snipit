# snipit

```javascript

module.exports  = (app)=>{
    // create a new link
    app.post("/api/link/new", function(req,res){
        db.Link.create({
            title: req.body.title,
            url: req.body.url,
            shortenedUrl: req.body.shortenedUrl,
            description: req.body.description,
            shared: req.body.shared,
            top500: req.body.top500,
            UserId: req.body.UserId
        }).then(function(dbLink){
            res.json(dbLink);
        })
    })
    // find all links in database
    app.get("/api/link/data", (req,res)=>{
        db.Link.findAll({
            include:[db.User]
        })
        .then((data)=>{
            res.json(data) // will be edited to not display user password

            })
    })
    // find all the information for a specific link
    app.get("/api/link/:id",(req,res)=>{
        db.Link.findOne({
            include:[db.User],
            where:{id:req.params.id,
                }

        }).then((data)=>{
            res.json(data)// will be edited to not display user password
        })
    })
    // delete a link
    app.delete("/api/link/delete", (req,res)=>{
        db.Link.destroy({
            where:{id:req.body.id}
        }).then((data)=>{
            res.json(data)
        })
    })
    // update a link
    app.put("/api/link/update", (req,res)=>{
        db.Link.update(
            req.body,
            {where:{id: req.body.id}}
        ).then((data)=>{
            res.json(data)
        })
    })
}

```

```javascript

class App extends Component {
  render() {
    return (
      <div>
        <ApolloProvider client={client}>
        <Router history={history}>
          <div>
          <Route exact path="/" render={(props) =>(
            !auth.isAuthenticated() ? (
              <Landing auth={auth} {...props} />
            ) : (
              <Redirect to="/home"/>
            )
          )} />
            <Route
              path="/"
              render={(props) =><Navbar auth = {
              auth
            }
            {
              ...props
            } />}/>
            <Switch>
              <Route exact path="/home" render={(props) =>(
                !auth.isAuthenticated() ? (
                  <Redirect to="/"/>
                ) : (
                  <Home auth={auth} {...props} />
                )
              )} />
              <Route exact path="/search" render={(props) =>(
                !auth.isAuthenticated() ? (
                  <Redirect to="/"/>
                ) : (
                  <Search auth={auth} {...props} />
                )
              )} />
            </Switch>
            <Route
              path="/callback"
              render={(props) => {
              handleAuthentication(props);
              return <Callback {...props}/>
            }}/>
          </div>
        </Router>
        </ApolloProvider>
      </div>
    );
  }
}

```

```javascript

const passport = require("./config/passport");
//Passport uses the concept of strategies to authenticate requests.
const PORT = process.env.PORT || 8080;
//AWS link to our bucket
const S3_BUCKET = process.env.S3_BUCKET;
const db = require("./models");

const app = express();
app.use(bodyParser.urlencoded({
  extended: true
}));
app.use(bodyParser.json());
app.use(express.static("public"));
app.use(session({
  secret: "keyboard cat",
  resave: true,
  saveUninitialized: true
})); //session middleware init
app.use(passport.initialize());
app.use(passport.session());

const exphbs = require("express-handlebars");
//handlebars init
app.engine("handlebars", exphbs({
  defaultLayout: "main"
}));
app.set("view engine", "handlebars");

//require("***routes.js")(app);

// heroku boilerplate GET route to configure our S3 bucket
app.get('/sign-s3', (req, res) => {
  const s3 = new aws.S3();
  const fileName = req.query['file-name'];
  const fileType = req.query['file-type'];
  const s3Params = {
    Bucket: S3_BUCKET,
    Key: fileName,
    Expires: 60,
    ContentType: fileType,
    ACL: 'public-read'
  };

  s3.getSignedUrl('putObject', s3Params, (err, data) => {
    if (err) {
      console.log(err);
      return res.end();
    }
    const returnData = {
      signedRequest: data,
      url: `https://${S3_BUCKET}.s3.amazonaws.com/${fileName}`
    };
    res.write(JSON.stringify(returnData));
    res.end();
  });
});

// This is our scraper route which will scrape the Moz top500 and return the results as JSON in a {0: url, 1: url} format
app.get('/scrape/', (req, res) => {
  let url = 'https://moz.com/top500';
  request(url, function (error, response, html) {
    let siteData = {};
    let $ = cheerio.load(html);
    $('td.url').each(function (i, elem) {
      siteData[i] = [];
      for (let j = 0; j < $(elem).children('a').length; j++) {
        siteData[i].push($(elem).children('a').text().trim());
      }
    });
    return res.json(siteData);
  });
});

// function that creates a request to our scraper route using a promise, it then parses the data and in a for loop creates entries into the db
const scraper = () => {
  rp('https://getlinkup.herokuapp.com/scrape/').then(function (res) {
    if (res != '') {
      let top500 = res;
      top500 = JSON.parse(top500);
      let size = Object.keys(top500).length;
      db.Top500.destroy({
        where: {}
      }).then(function () {
        for (let i = 0; i < size; i++) {
          let url = top500[`${i}`][0];
          db.Top500.create({
            url: url
          }).catch(function (err) {
            console.log(err);
          });
        }
      });
    }
  });
}

const top500Validation = (linkToCheck, cb) => {
  $.get('/scrape/', function (data) {
    if (data != '') {
      let top500 = data;
      top500 = JSON.parse(top500);
      let size = Object.keys(top500).length;
      let result = false;
      for (let i = 0; i < size; i++) {
        let url = top500[`${i}`][0];
        if (linkToCheck == url) {
          result = true;
          return result
        }
      }
      top500 = result;
    }
  })
}

//this cronjob is set to run every day at midnight to reset the dailyClicks in the db
schedule.scheduleJob('00 00 * * *', function () {
  console.log('Time to clear the daily clicks!');
  db.Link.update({
    dailyClicks: 0
  }, {
    where: {}
  });
});

db.sequelize.sync().then(() => {
  app.listen(PORT, () => {
    console.log("==> 🌎  Listening on port %s. Visit http://localhost:%s/ in your browser.", PORT);
  });
});

```

```javascript

var bandIs = function (){

  var name = window.location.href;
  var thing= name.split("/")
  var lastUrl= thing[thing.length-1];

    var bandQuery = lastUrl;
    //concert info
    var queryURL = "https://rest.bandsintown.com/artists/" + bandQuery + "/events?app_id=bandit";
    //artist info
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
    //  band in town api
}

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

```
