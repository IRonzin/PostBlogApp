# go-gin-redis-mongodb

![Diagram](https://raw.githubusercontent.com/jeff-vincent/go-gin-redis-mongodb/dev/images/Untitled%20Diagram-32.drawio.svg)

The Web API has routes for uploading and viewing blog posts, as well as a route to get the number of times a given post has been viewed. 

When a post is uploaded, it is published to the Redis channel "Upload," to which the db_worker is subscribed. When the db_worker gets the message from Redis, it inserts the post into Mongo1. 

When a post is viewed, the Web API passes the request to the Blog Service, which then queries Mongo1 for the requested title. If it is found, before the post is returned to the Web API, the Blog Service publishes the title to the "Analytics" channel in Redis. This way, the Analytics Worker can asynchronously check to see if the post has been viewed before -- if it has, the number of views is increased by 1, if it hasn't, the title is entered into the Mongo2 instance for analytics tracking. 

Finally, the Web API also has a route to handle a request for the number of times a given post has been viewed, which it forwards to the Analytics Service, which then queries Mongo2 and returns the result. 

kubectl logs web-api-7cd95d6d9c-5srs4 - логи
kubectl get ingress - ингресс
kubectl get svc - сервисы Kuber
minikube -n kong service kong-proxy - проксирование эндпоинта в локалхост (нужно только для миникуба)
kubectl get svc -n kong - сервисы Kuber в пространстве kong (ingress)
helm template . --values values.yaml | kubectl apply -f - //// последняя тире нужна - команда для обновления микросервисов через helm
protoc --go_out=C:\Users\VladimirRonzin\source\repos\PostBlogApp\go-gin-redis-mongodb\redis-gin-mongo\src\db_worker .\db_worker.proto - генерация структур proto

https://velocity.tech/blog/build-a-microservice-based-application-in-golang-with-gin-redis-and-mongodb-and-deploy-it-in-k8s

1. web-api sends a new post to db-worker with Redis queue using Protobuf (proto3)


