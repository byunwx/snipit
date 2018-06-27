# snipit
## if the string on process.argv[2] is palindrome console.log palindrome if not not palindrome
```javascript
var string = ""
if (process.argv[2]) {
  string=process.argv[2];
}
var stringHalf = 10;
var count= 0;

if (string=="") {
  console.log("You need to write something")
}else if (string.length%2 == 0) {
  stringHalf= string.length/2-1;
  for (var i = 0; i < stringHalf; i++) {
    if(string[i]==string[string.length-1-i]){
      count+=1;
    }
  }
} else {
  stringHalf= (string.length-1)/2;
  for (var i = 0; i < stringHalf; i++) {
    if(string[i]==string[string.length-1-i]){
      count+=1;
    }
  }
}
if (count==stringHalf && string.length>1) {
  console.log(process.argv[2]);
  console.log("is palindrome");
}else if(string.length==1){
  console.log("You only have one letter")
}else{
  console.log(process.argv[2]);
  console.log("is not palindrome")
}
```
## Write a function that outputs an array of the nth Fibonacci number. 
```javascript
var processNum=[0, 1];

var n= process.argv[2];
if (process.argv[2] && n%1==0) {
if(n>1){
  for (var i = 2; i <= n; i++) {
    var added= processNum[i-1]+processNum[i-2];
    processNum.push(added);
  }
  console.log(processNum.join(" + "))
  console.log(processNum[n]);
}else if(n==0){
  console.log("0");
  console.log(processNum[n]);
}else{
  console.log(processNum.join(" + "))
  console.log(processNum[n]);
}
} else{
  console.log("need to write a number")
}

```

## DateNight search component 

```javascript
/*dependencies*/
import React, {Component} from "react";
import MapView from '../mapView/mapView';
import Input from './input'
import { ApolloConsumer } from "react-apollo";
import {GET_YELP_RESULT, CREATE_ITINERARY} from './queries'
import {Modal, Button} from 'react-materialize'
import {Mutation} from 'react-apollo'

// Jon TODO: add remove/update itinerary buttons w/functionality, deal with the modal
class Search extends Component {
  state = {
    yelpSearch: null,
    search: '',
    location: '',
    name: '',
    date: '',
    time: '',
    currentItinerary: [],
    itineraries: [],
    profile:{}
  }
  componentWillMount() {
    const { userProfile, getProfile } = this.props.auth;
    if (!userProfile) {
      getProfile((err, profile) => {
        this.setState({ profile });
      });
    } else {
      this.setState({ profile: userProfile });
    }
  }

  onYelpFetched = x => this.setState({ yelpSearch: x })

  handleInputChange = event => {
    const {name, value} = event.target;
    this.setState({[name]: value});
  }

  render() {
    console.log(this.state.profile);
    return (
      <div>
         <video autoPlay muted id="homeVideo">
                    <source src='http://www.coverr.co/s3/mp4/Broadway.mp4'
                        type="video/mp4" />
                    </video>
        {/*  Left Column
                        tabs: upcoming | Planning | Past
                        Render array of itins*/}
        <div className="row container content">
          <div className="sidebar col s12 m3 ">
            <div className="row">
            <h3 className="center-align">Search</h3>
              {/* <p>Search tabs router goes here</p> */}
              <form>
                <Input className="main-content"
                  onChange={this.handleInputChange}
                  name="search"
                  placeholder="Search"/>
                <Input
                  onChange={this.handleInputChange}
                  name="location"
                  placeholder="Location (Zip, Address, City)"/>
                  <ApolloConsumer>
                    {client => (
                      <div
                        className=' search-page-btn hoverable waves-effect waves-light btn-small center-align'
                        onClick={async() => {
                        const {data} = await client.query({
                          query: GET_YELP_RESULT,
                          variables: {
                            search: this.state.search,
                            location: this.state.location
                          }
                        })
                        this.onYelpFetched(data.yelpSearch)
                      }}>
                        Search
                      </div>
                    )}
                  </ApolloConsumer>
              </form>
            </div>
              {this.state.yelpSearch
                ?
                this
                  .state
                  .yelpSearch
                  .map(({_id, name, location, url, price, phone, coordinates}, i) => (
                    <div key={_id}>
                      <h6>
                        <a  href={`${url}`} target="_blank">{`${i + 1} ${name}`}</a>
                        {` ${price}`}
                      </h6>
                      <p>{`${location}`}</p>
                      <p>{`${phone}`}</p>
                      <div className='btn-small hoverable search-page-btn' onClick={async ()=>{
                        const itinItem = {
                          _id: this.state.currentItinerary.length + 1,
                          name: name,
                          location: location,
                          url: url,
                          price: price,
                          phone: phone,
                          coordinates: coordinates
                        }
                        await this.setState({currentItinerary: [...this.state.currentItinerary, itinItem]})
                      }}>Add</div>
                    </div>
                  ))
                : <p>Results will appear here after you hit search!</p>}
            </div>
          <div className="main-content col s12 m3">
            <h3 className="center-align">Itinerary</h3>
            {this.state.currentItinerary.length > 0 ? this.state.currentItinerary.map(({name, location, url, phone}, i) => (
                    <div key={url}>
                      <h6>
                        <a href={`${url}`} target="_blank">{`${name}`}</a>
                      </h6>
                      <p>{`${location}`}</p>
                      <p>{`${phone}`}</p>
                      <div className='btn-small search-page-btn' onClick={async ()=>{
                        await this.setState((prevState) => ({
                          currentItinerary: prevState.currentItinerary.filter((_, j) => j !== i)
                        }))
                      }}>Remove</div>
                    </div>
            )) : <p>Your Current Itinerary will appear here once you have added something to it</p>}
            {this.state.currentItinerary.length > 0 ?
            <Modal
                header={<h5>Review Itinerary</h5>}
              trigger={<Button className="btn-small finalize-btn search-page-btn">Name This Date</Button>}
              >
                <Input
              className="result-name"
                onChange={this.handleInputChange}
                name="name"
                placeholder="Name your Itinerary"/>
              <Input
              className="result-body"
               onChange={this.handleInputChange}
               name="date"
               type="date"
               placeholder=""/>
                <Input
                className="result-body"
                onChange={this.handleInputChange}
                name="time"
                type="time"
                placeholder=""/>
              {this.state.currentItinerary.map(({name, location, url, phone}, i) => (
                <div key={url}>
                  <h6>
                    <a className="result-name" href={`${url}`} target="_blank">{`${name}`}</a>
                  </h6>
                  <p className="result-body">{`${location}`}</p>
                  <p className="result-body">{`${phone}`}</p>
                  <div className='btn' onClick={async ()=>{
                        await this.setState((prevState) => ({
                          currentItinerary: prevState.currentItinerary.filter((_, j) => j !== i)
                        }))
                      }}>
                     Delete
                  </div>
                </div>
              ))}
            <Mutation mutation={CREATE_ITINERARY}
            onCompleted={(data)=>{
            const activities = data.createItinerary.activities.map(x => {
              const presentable =`
              Name: ${x.name}
              Location: ${x.location}
              Phone: ${x.phone}`
               return presentable
              }
              )
            alert(`     Itinerary Created!
            Name: ${data.createItinerary.name}
            Date: ${data.createItinerary.date}
            Time: ${data.createItinerary.time}
            Activities: ${activities}`
            )}}>
            {(createItinerary, error) => (
            <Button
              className="btn-large finalize-btn" onClick={async e => {
                e.preventDefault()
                if (this.state.name === '' || this.state.date === '' || this.state.time === '' || this.state.currentItinerary.length === 0) {
                  return alert('Please fill out all fields')
                }
                console.log(this.state.profile.sub);
                await createItinerary({ variables: { name: this.state.name, date: this.state.date, time: this.state.time, activities: this.state.currentItinerary, userauth: this.state.profile.sub}})
                if (error.error !== undefined) {
                  console.log(error.error)
                }
              }}>
              Finalize
            </Button>
            )}
            </Mutation>
            </Modal>
            : ''}
            </div>
            {/* end of Itinerary code */}
            <div className="col s12 m3">
              {/* <h2 align="center">Map</h2> */}
                <MapView className=" map" yelpSearch={this.state.yelpSearch} currentItinerary={this.state.currentItinerary}/>
          </div>
        </div>
      </div>
    )
  }
};

export default Search;

```
