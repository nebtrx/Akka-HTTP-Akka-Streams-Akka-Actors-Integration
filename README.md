# Akka HTTP + Akka Streams + Akka Actors Integration Example
This application is able to respond with a Streaming JSON response of 
Chuck Norris jokes taken from [here](http://www.icndb.com/api/)

The Internet Chuck Norris Database returns a discrete JSON response for
every HTTP request sent out. Whilst this is okay for most cases, it 
becomes a bit cumbersome if you want to look at Chuck Norris jokes 
as you have to keep hitting the endpoint yourself. The goal of this 
project is to turn those discretized JSON responses into a continuous
JSON stream. This is done with the help of Akka HTTP, Akka Actors and 
Akka Streams. 

The `JokeFetcher` Akka Actor is used to continuously poll the ICNDB 
endpoint behind the scenes and publish each discrete JSON response onto
the Event Stream as a JokeEvent. 

The `JokePublisher` Akka Actor is a point of integration between Akka 
Actors and Akka Streams. The `JokePublisher` is created whenever a 
request to the `/streaming-jokes` endpoint which in turn creates an Akka 
Stream to stream the response back to the requester. It lives for the 
duration of that Stream and gets terminated once the Stream completes 
(does not happen in this case because it is an infinite Stream) or when 
the Stream is cancelled (happens when the requester does not want 
anymore data). As you can see the JokePublisher is tied to the duration 
of each Stream. This means if 6 users come in and request data from 
`/streaming-jokes` then 6 actors of JokePublisher will be created, each
of them will be responsible for publishing data into that specific 
user's streaming response. The `JokePublisher` on creation, will 
subscribe to the Akka EventStream and listen for `JokeEvent`s and send
them downstream to provide streaming responses. As you can see, 
publishing messages to the `JokePublisher` Actor will end up in the 
Akka Stream which is why this is a point of integration between Akka 
Actors and Akka Streams when it comes to publishing data into the Akka 
Stream. This actor respects backpressure. It will only send information 
as fast as the downstream consumer can consume. It will drop messages if 
they are coming in too quickly. 

Users can hit the `/streaming-jokes` route in order to get back an [SSE](http://www.html5rocks.com/en/tutorials/eventsource/basics/)
streaming JSON response of Chuck Norris jokes in JSON format.

### Pre-requisites:
- Scala 2.11.8
- [SBT](http://www.scala-sbt.org/)

### Instructions: 
- `sbt run` to start the application
- Visit `localhost:9000/streaming-jokes` to see the Streaming response

## Credits:
- [Akka](http://akka.io)
- Heiko Seeberger's [Akka SSE extension](https://github.com/hseeberger/akka-sse)
- Lomig Megard's [Akka HTTP CORS extension](https://github.com/lomigmegard/akka-http-cors)