# Mini_microservice
A "mini" microservice, consisting of a react app and five express servers.

One kubernetes cluster configured to run with Skaffold.

## Build locally
Make sure you have Docker and Kubernetes installed.

Then you have to make posts.com redirect to 127.0.0.1.
This is done in your `etc/hosts` file. Add: `127.0.0.1 posts.com` at the end of the file.

Then you'll have to install [Skaffold](https://skaffold.dev/). Instrunctions can be found [here](https://skaffold.dev/docs/install/).

Navigate to the root folder and run `skaffold dev`. This will set up the kubernetes cluster.

If you then go to posts.com you will be able to create posts and comments. NB: Comments and posts are saved in the memory of the running services, so they will be lost when shutting down the pods.

## Architecture
The microservice arhitecture is based on an async architecture with one event-bus. All events are broadcasted to all services.

The kubernetes cluster consists of a load-balancer, ingress and 5 pods. The load-balancer and ingress was created with [nginx-ingress](https://kubernetes.github.io/ingress-nginx/). The different pods communicates with ClusterIP.

### The different services:
* Posts: Create posts. Doesn't care for events from the bus. 
* Comments: Create comments. Comments can have three statuses: `pending`, `rejected`, `approved`. Initially the status is set to `pending`. The comment service listens to `CommentModerated` and emits `CommentUpdated` when a comment has been moderated, and `CommentCreated` when a comment is initially created. 
* Moderation: Moderates all the comments. Any comment that contains the word `orange` will be rejected. The moderation service listens for `CommentCreated` and emits `CommentModerated` after moderation.
* Query: Have endpoint to return all posts with comments. Listen for the current events to maintain the query: `PostCreated`, `CommentCreated` and `CommentUpdadet`.
* EventBus: Broadcasts all events. Saves all events in memory, making them available for services that crashes/restarts.
* Client: Serves the react-app.
