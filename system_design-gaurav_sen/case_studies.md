# Case Studies for system design

# Netflix 

Processing Video:
- The video uploaded by the user is processed and stored based on format and resolution. 
- The video format decides the video quality(High, medium, low) - mp4, avi, etc.
- The video resolutions are 320, 480, 720, 1080, etc. 
- The number of tuples formed are Formats * Resolutions. That is the number of videos netflix ends up processing and storing. 
- Processing complete video may take a lot of time. Therefore the video is broken down in pieces and then processed. this will result in (a.mp4, 480p), (a.avi, 720p), (b.mp4, 720p), etc.  
- The breaking down is done based on scenes rather than based on timestamps to improve user experience. 
- Breaking down the video also helps to mitigate the problem of system failure while processing the video. It minimizes the computation loss.

Streaming Video:
- Every video is played based on the device and the internet connection available to the user. 
- Every movie is broken down in pieces based on scenes, and broken down further in smaller parts. 
- If a movie is sparsely watched, when the user asks for a randomly selected part, it only asks for that small part which is asked for.
- If a movie is densely watched, the complete scene is downloaded to improve user experience. 

The videos are stored in AWS S3. 

Caching:
- All the ISPs have a cache box. 
- These ISPs cache the frequently seen videos based on locations so that new requests dont have to fetch the movie from US servers, thus improving the user experience.
- 90% of the traffic is served by these local ISPs, thus making it much faster.
- We can update the cache box with new movies at a time with minimal load, for ex at 4 pm. 

# Tinder

When designing systems, instead of trying to start with data models, then going to services, then to client requests, follow the other way round. Start with the features required by the customers. 

Features: 
- Storing Profiles ( Images, 5 per profile )
- Recommend Matches (No of active users)
- Note matches (around 0.1% of swipes)
- Direct Messaging (chat system)


### Storing Images

2 ways to store: 
- file 
- Blob (Binary large object)

Blob stored in DB. DB provides us following features:
- Mutability - Not required, as not gonna change the image partially
- Transaction Gurantees(ACID) - NR, as no transaction on images
- Indexes (faster Search) - but no search inside image
- Access Control - same thing can be achieved with File system as well

Pros of File System:
- Cheaper
- Faster
- can build CDNs for faster access. 

Images will be stored in Distributed File System. 

 - structure of stored data. 

#### Architecture:
 - **Gateway** - only service talking to the client. takes a request, asks profile server if request is authenticated, if yes sends it to correct service and send back the response to the client. 
 - **Profile service** : Basic functionalities like registering a user (will require mail service). After that authenticate requests(using Tokens). Stores basic data like name, desc, etc of the user in a Database.
 - **Image service** : Images are stored on a Distributed File System, handled by a seperate service, as images can be used by other services (except profile) as well (like for ML, data analytics). Following data will be stored in a attached database -  | profile ID | Image ID | File Url |. 

## Chat Service 

Protocol - XMPP for 2 way communication (chat). With HTTP will require polling, inefficient.

### Architecture
- Open websockets connections (TCP for simplicity) with users
- A seperate session service to store user id and connection Ids.
- A matching service to check if the user1 is allowed (matched or not) to send the message to user2.
- receive message, check if allowed, find connection id of user2, send the message via socket.
- Also store the chat messages in a database based on pairs of users.



# Whatsapp 

