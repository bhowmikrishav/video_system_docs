# File Store System

The Video System requires a microservices which manages and handles IO operation for media files and object storage. We call this microservice File Store System.

Filestore system workes with RESTFUL endpoints or WebSockets, depending on the respective IO operation taking place.

### RESTFUL Endpoints

These endpoints intended to deal with files/object is size up to 2GB. However, to ensure the best-practices limit is assigned 32MegaBytes.

#### Best Practices

- Reduce/Eliminate any IO bottlenecks on the DB
- Reduce query time
- Prevent need of a single node to be very large
- Follow a streaming mechanism large files
