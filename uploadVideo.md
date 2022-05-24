# Context
## Demand
The company needs to realize a page for users to upload their videos online. In the face, the user flow is:
+ click `select` button
+ choose a video
+ upload the video online
<img width="940" alt="image" src="https://user-images.githubusercontent.com/75397936/170137117-ae9ec9d4-9195-4b75-afc4-492c263ea2f8.png">

## Environment
Stream Video API we are using is `Mux`. So videos should upload into `Mux`. 
For general data storage, firestore is chosen. Here, `video` data should be stored in the `video` collections. For each video, there will be a document for record.
In addition, videos cannot store in the web server. In future, the videos will be uploaded in some long-term storage database.
<img width="922" alt="image" src="https://user-images.githubusercontent.com/75397936/170136715-8ef52907-f8ec-401a-8894-0106d3ba0d30.png">

## Logic
<img width="989" alt="image" src="https://user-images.githubusercontent.com/75397936/170136852-fb33af29-57aa-44f3-aedb-9ad8f098ff71.png">

Transaction is just my thought and doubt: how to implement transcation to group some operations together? Still no answer


# Code
